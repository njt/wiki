---
url: https://ranvier.systems/2026/04/30/kv-cache-locality-the-hidden-variable-in-your-llm-serving-cost.html
date_fetched: 2026-07-05
backfilled: true
---

# KV Cache Locality: The Hidden Variable in Your LLM Serving Cost

Every time your load balancer sends a request to the wrong GPU, that GPU recomputes a prefill it already computed somewhere else. The KV cache for that 4,000-token system prompt exists. It’s just sitting on a different card. Your load balancer doesn’t know. It can’t know. It’s counting connections, not tokens.

That recomputation takes real time and real money. On a Llama 3.1 70B at half precision, a 4,000-token prefill takes over a second. If eight GPUs each recompute the same system prompt independently because round-robin sent one request to each, you just paid for the same work eight times. Multiply by every request, every hour, every day.

This post is about the cost of that mistake, how to measure it, and what changes when your load balancer understands token locality.

## What the KV Cache Actually Saves You

A transformer processes input tokens in two phases. **Prefill** computes the
key-value pairs for every input token: the system prompt, the conversation
history, the RAG context. This is the expensive part. It scales with token
count and model size, and it’s compute-bound on the GPU. **Decode** generates
output tokens one at a time, each one reusing the key-value pairs from
prefill. This is the cheap part.

vLLM and other serving engines cache the key-value pairs from prefill in GPU memory. When a new request arrives with the same token prefix, the engine skips prefill entirely and jumps straight to decode. This is the KV cache hit.

On our benchmarks, a cache hit on CodeLlama 13B returns in 18ms at P50. A cache miss takes around 500ms. That’s a 28x gap in time-to-first-token, decided entirely by whether the tokens were already on that GPU.

But here’s the thing: the KV cache is **per-GPU**. GPU 0’s cache doesn’t help
GPU 3. If your load balancer sends Request A to GPU 0 and the identical
Request B to GPU 3, Request B pays full prefill cost even though the work was
already done. The cache exists. It’s just on the wrong card.

## The Math on Wasted Prefill

Let’s make this concrete. You’re running a RAG application with a 4,000-token system prompt. You have 8 GPUs serving CodeLlama 13B. You’re handling 30 concurrent users with a stress workload (heavy on large and extra-large prefixes). Here’s what we measured on 8x A100s:

Round-robin routing:

- Cache hit rate: 12.5%
- P99 TTFT: 6,800ms
- Throughput: 36.3 req/s

With 8 backends and random routing, you’d expect ~12.5% cache hits by chance. One in eight requests happens to land on the GPU that already has its prefix cached. The other 87.5% recompute from scratch.

Prefix-aware routing:

- Cache hit rate: 97.5%
- P99 TTFT: 1,000ms
- Throughput: 44.4 req/s

Same GPUs. Same model. Same workload. The only change is which GPU receives which request.

That throughput difference, 36.3 vs 44.4 requests per second, is a 22.3% improvement. On hardware costing ~$10/hour, that’s either 22% more throughput for free or the same throughput on fewer GPUs. Over a month of continuous operation, on a single 8-GPU node, the wasted prefill in round-robin comes to roughly $1,200–$1,800 in GPU-hours (22% of ~$7,300/month at $10/hr) that produce no useful work. Multiply by the number of nodes in your cluster.

## Where the Savings Compound

The benefit scales with three variables: **model size**, **prefix length**,
and **prefix sharing ratio**.

### Model size

Larger models have more expensive prefill, so cache misses cost more.

| Model | XLarge Cache Hit Improvement | Aggregate Throughput Gain | 
|---|---|---|
| Llama 3.1 8B | 31.6% | ~0% (inference too fast) | 
| CodeLlama 13B | 35.9% | +13.7% to +22.3% | 
| Llama 3.1 70B | 43.8% | ~0% (compute-bound) | 

The 8B numbers are the warning case. When prefill is already fast (~420ms total inference), the 7-10ms routing overhead eats into the savings. If prefill isn’t your bottleneck, prefix-aware routing doesn’t help.

The 70B numbers tell a different story. Aggregate throughput doesn’t change because the GPUs are already compute-saturated. But individual requests are 44% faster on cache hit (P50: 1,498ms hit vs 2,665ms miss). Your users feel the difference even if your throughput dashboard doesn’t.

The sweet spot is 13B-70B models where prefill is expensive enough to matter but the GPUs aren’t so saturated that they can’t benefit from skipping it.

### Prefix length

Longer shared prefixes mean more wasted compute per cache miss.

| Max Prefix Tokens | Cache Miss P50 | Cache Hit P50 | Improvement | 
|---|---|---|---|
| 8,192 (default) | 638ms | 448ms | 29.7% | 
| 16,384 | 817ms | 461ms | 43.6% | 

At 16K tokens, a cache miss wastes nearly 400ms of GPU compute that a hit avoids entirely. As context windows keep growing, this gap widens.

### Prefix sharing ratio

This is the percentage of tokens shared across requests. A RAG application where every request includes the same 4,000-token knowledge base has a high sharing ratio. A chat application where every conversation is unique has a low one.

| Sharing Ratio | Round-Robin Hits | Prefix-Aware Hits | Improvement | 
|---|---|---|---|
| 50% | ~11% | 91% | +80pp | 
| 70% | ~13% | 90% | +77pp | 
| 90% | ~12% | 97-98% | +85pp | 

Even at 50% sharing, where half the tokens are unique, prefix-aware routing still achieves 91% cache hits. A consistent hash fallback (deterministic routing based on prefix when no learned route exists yet) ensures that requests with the same prefix land on the same GPU even before the system has observed them.

## The P99 Story

Cost isn’t just GPU-hours. It’s also the cost of slow responses.

At 30 concurrent users on CodeLlama 13B over 30 minutes of sustained load, round-robin routing produced a P99 TTFT of 6,800ms. That’s 6.8 seconds before the first token appears. For an interactive application like code completion or chat, that’s a broken experience. Users don’t wait 6.8 seconds.

Prefix-aware routing brought that same P99 down to 1,000ms. Same hardware, same model, same concurrency. An 85.3% improvement on tail latency.

Why does the tail improve so much? Because tail latency in LLM serving is driven by cache misses under load. When the GPU is busy generating tokens for other requests, a new request that requires full prefill gets queued behind them. With round-robin, 87.5% of requests need full prefill, so the queue is always full of expensive work. With prefix-aware routing, 97.5% of requests skip prefill entirely, so the queue drains faster and the few remaining misses get processed sooner.

This is the strongest argument for KV cache locality. Throughput improvements look good on a dashboard. Tail latency is what users actually experience.

## What Doesn’t Work

Prefix-aware routing isn’t free, and it doesn’t help everywhere.

**Small models (≤8B):** Inference is already fast enough that the routing
overhead (~10ms for tokenization + tree lookup) approaches the prefill
savings. The net effect is roughly zero.

**Short prefixes (<500 tokens):** The prefill cost for short sequences is
small enough that cache misses don’t meaningfully hurt. The routing overhead
(~3ms minimum) can exceed the savings.

**Unique conversations:** If every request has a completely different prefix
(no shared system prompt, no shared context), there’s nothing to cache. The
routing tree learns routes that are never reused.

**Load imbalance:** Strict prefix affinity can create hot spots. If 80% of
your traffic shares the same system prompt, prefix-aware routing sends 80% of
traffic to one GPU. We handle this with a load-aware fallback that diverts
requests when a backend’s in-flight count exceeds twice the median. This
trades a cache miss for a balanced GPU, reducing P95 by 36% and P99 by 45%
compared to strict affinity. The cache hit rate drops about 5 points, which
is the right trade.

## Measuring Your Own Cache Locality

Before you change anything, measure your current cache hit rate. Most vLLM deployments expose this via Prometheus:

- `vllm:gpu_prefix_cache_hit_rate`(or- `vllm:gpu_prefix_cache_queries_total`and- `_hits_total`on older versions; check your- `/metrics`endpoint)
- Compare TTFT distributions between requests with shared vs unique prefixes
- Look at your P99/P50 ratio. A ratio above 5x suggests cache thrashing

If your cache hit rate is already above 80%, you’re either lucky or your traffic naturally clusters. If it’s below 30%, you’re leaving performance on the table.

The variables that matter most:

- **How many GPUs are you routing across?**More GPUs = lower chance of random cache hits. With 8 GPUs, random routing gives ~12.5% hits.
- **How long are your shared prefixes?**Longer = more wasted compute per miss.
- **What’s your prefix sharing ratio?**Higher = more opportunity for reuse.
- **What model size are you serving?**Larger = more expensive prefill per miss.

If you have many GPUs, long shared prefixes, high sharing ratios, and large models, you’re likely wasting 20-40% of your GPU compute on redundant prefill.

## The Takeaway

KV cache locality is not a tuning knob. It’s a multiplier on your existing hardware. The same GPUs, serving the same model, handling the same traffic, produce measurably different throughput and latency depending on one decision: which GPU gets which request.

Round-robin doesn’t make that decision. Least-connections doesn’t make that
decision. They balance load without understanding what the load *is*. When
every request carries thousands of tokens that might already be cached
somewhere in your cluster, “balanced” and “efficient” are not the same thing.

We built Ranvier to make that decision. It routes requests to the GPU that already has their token prefix cached, using an adaptive radix tree that learns routes in real time. The first post in this series covered why your load balancer is wasting your GPUs. This post covered what that waste costs. The next one will cover how we tokenize 50,000 requests per second without blocking the event loop.

*All benchmarks run on 8x A100 GPUs (Lambda Labs), February 2026. Workloads
use the stress distribution (10% small, 20% medium, 30% large, 40% xlarge
prefixes) with 90% prefix sharing ratio unless noted. Full methodology and raw
data available in the
benchmark guide.*

*Ranvier is a project of Minds Aspire, LLC.*
