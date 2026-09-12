---
url: https://ranvier.systems/2026/04/30/kv-cache-locality-the-hidden-variable-in-your-llm-serving-cost.html
title: "KV Cache Locality: The Hidden Variable in Your LLM Serving Cost"
author: Minds Aspire (Ranvier project)
date_fetched: 2026-05-18
date_published: 2026-04-30
topics:
  - ai-infrastructure-and-hardware
---

# KV Cache Locality: The Hidden Variable in Your LLM Serving Cost

**Author:** Minds Aspire (Ranvier project)
**Published:** April 30, 2026
**Platform:** Ranvier blog

---

## Core Thesis

When a load balancer routes a request to a GPU that lacks the relevant KV cache, the system recomputes a prefill that already exists on a different card. "Your load balancer doesn't know. It can't know. It's counting connections, not tokens."

---

## KV Cache Mechanics

Transformers process tokens in two phases:

- **Prefill** — Computes key-value pairs for all input tokens (system prompt, conversation history, RAG context). This is compute-bound and expensive.
- **Decode** — Generates output tokens one at a time, reusing precomputed KV pairs. This is cheap.

When a new request shares a token prefix with a previously cached one, serving engines like vLLM skip prefill entirely — a **KV cache hit**.

The critical constraint: KV caches are per-GPU. GPU 0's cache is useless to GPU 3.

### Benchmark: CodeLlama 13B
- Cache hit P50: **18ms**
- Cache miss P50: **~500ms**
- That's a **28x gap** in time-to-first-token (TTFT).

---

## The Cost of Misdirected Traffic

### Round-robin routing (8 GPUs, CodeLlama 13B, 30 concurrent users):
- Cache hit rate: **12.5%**
- P99 TTFT: **6,800ms**
- Throughput: **36.3 req/s**

### Prefix-aware routing (same hardware, same workload):
- Cache hit rate: **97.5%**
- P99 TTFT: **1,000ms**
- Throughput: **44.4 req/s**

That's a **22.3% throughput gain**. On an 8-GPU node at ~$10/hour, wasted prefill under round-robin costs roughly $1,200–$1,800/month in GPU-hours — 22% of ~$7,300/month. Scale that across a multi-node cluster.

---

## Where Savings Compound

### By model size

| Model | Cache Hit Improvement | Throughput Gain |
|---|---|---|
| Llama 3.1 8B | 31.6% | ~0% (inference too fast) |
| CodeLlama 13B | 35.9% | +13.7% to +22.3% |
| Llama 3.1 70B | 43.8% | ~0% (compute-bound) |

The 8B case is a warning: when prefill takes ~420ms total, routing overhead eats savings. The 70B case shows no aggregate throughput gain because GPUs are already saturated — but individual requests are 44% faster on cache hit, so users feel the difference even if dashboards don't show it. The sweet spot is **13B–70B**.

### By prefix length

| Max Prefix Tokens | Miss P50 | Hit P50 | Improvement |
|---|---|---|---|
| 8,192 | 638ms | 448ms | 29.7% |
| 16,384 | 817ms | 461ms | 43.6% |

Longer contexts widen the gap. At 16K tokens, a miss wastes nearly 400ms of GPU compute.

### By prefix sharing ratio (percentage of tokens shared across requests)

| Sharing Ratio | Round-Robin Hits | Prefix-Aware Hits | Improvement |
|---|---|---|---|
| 50% | ~11% | 91% | +80 pp |
| 70% | ~13% | 90% | +77 pp |
| 90% | ~12% | 97–98% | +85 pp |

Even at 50% sharing, prefix-aware routing achieves 91% cache hits. A consistent hash fallback ensures requests with the same prefix land on the same GPU even before the system has observed them.

---

## The P99 Story

At 30 concurrent users on CodeLlama 13B over 30 minutes:

- Round-robin P99 TTFT: **6,800ms** (broken for interactive use)
- Prefix-aware P99 TTFT: **1,000ms**
- **85.3% improvement** on tail latency

Why? Tail latency in LLM serving is driven by cache misses under load. With round-robin, nearly 9 in 10 requests need full prefill, so the queue is perpetually full of expensive work. With prefix-aware routing, nearly all requests skip prefill, so queues drain faster and the few misses get processed sooner. The author calls this "the strongest argument for KV cache locality."

---

## What Doesn't Work

Prefix-aware routing has clear limitations:

- **Small models (≤8B):** Routing overhead (~10ms) approaches prefill savings; net effect is roughly zero.
- **Short prefixes (<500 tokens):** Prefill cost is too small to matter; routing overhead (~3ms minimum) can exceed savings.
- **Unique conversations:** No shared prefix means nothing to cache; the routing tree learns routes that are never reused.
- **Load imbalance:** Strict affinity can create hot spots. If 80% of traffic shares one system prompt, 80% goes to one GPU. The fix: a load-aware fallback that diverts requests when a backend's in-flight count exceeds twice the median. This drops cache hit rate about 5 points but reduces P95 by 36% and P99 by 45% — "the right trade."

---

## How to Measure Your Own Cache Locality

Use vLLM's Prometheus metrics:
- `vllm:gpu_prefix_cache_hit_rate` (or older `_queries_total` / `_hits_total` metrics)
- Compare TTFT distributions between shared vs. unique prefixes
- A P99/P50 ratio above **5x** suggests cache thrashing

### Key diagnostic questions:

1. **How many GPUs are you routing across?** More GPUs = lower random hit rate. With 8, random routing yields ~12.5%.
2. **How long are your shared prefixes?** Longer = more waste per miss.
3. **What's your prefix sharing ratio?** Higher = more opportunity.
4. **What model size?** Larger = more expensive prefill per miss.

If you have many GPUs, long shared prefixes, high sharing ratios, and large models, you're likely wasting **20–40% of GPU compute** on redundant prefill.

---

## Key Takeaway

KV cache locality "is not a tuning knob. It's a multiplier on your existing hardware." Round-robin and least-connections balance load without understanding what the load *is* — when every request carries thousands of potentially cached tokens, "balanced" and "efficient" are not the same thing.

The post concludes with a teaser for a follow-up on tokenizing 50,000 requests per second without blocking the event loop. The project is open-source on GitHub under the Ranvier name, by Minds Aspire, LLC.

**Benchmark context:** All runs on 8x A100 GPUs (Lambda Labs), February 2026, using a stress workload distribution (10% small, 20% medium, 30% large, 40% xlarge prefixes) with 90% prefix sharing ratio unless otherwise noted.
