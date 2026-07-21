---
url: https://www.oreilly.com/radar/the-tokens-you-cant-wait-for/
title: "The Tokens You Can't Wait For: Text diffusion, the GPU hangover, and the one place parallel generation actually pays off"
author: Shreshta Shyamsundar and Anmol Jain
date_fetched: 2026-07-21
date_published: 2026-07-20
site: O'Reilly Radar
---

# The Tokens You Can't Wait For

**Subtitle:** Text diffusion, the GPU hangover, and the one place parallel generation actually pays off

**Authors:** Shreshta Shyamsundar and Anmol Jain

**Published:** July 20, 2026 • 12 minute read

**Source:** O'Reilly Radar, filed under AI & ML

---

The piece opens with a vignette about a bank in Singapore that bought eight H100s for sovereign AI compute but now faces utilization problems — what the authors call "the GPU hangover." The hardware arrived, but utilization did not, because of a mismatch between how standard models generate text and how enterprises actually use them.

## The Physics Problem

Standard autoregressive models (Llama, Mistral, GPT) generate one token at a time. The weights sit in GPU high-bandwidth memory but must be "streamed out of that main memory and through the compute units again" for every single token, because on-chip memory can't hold a multibillion-parameter model. This creates a memory bottleneck rather than a compute bottleneck — arithmetic intensity sits near 1 at batch size one, while GPUs are built for intensities in the hundreds.

Batching is the escape hatch: read weights once and compute the next token for hundreds of requests simultaneously. Small vs. large batches can swing cost 10- to 30-fold on the same hardware. Overnight queues of millions of documents are trivially batchable. But single requests that must return in under a second — code completions, onboarding checks — can't wait to fill a batch. The first workload is not truly memory-bound; the second is.

A further subtlety: generating tokens is memory-bound, but reading the prompt is compute-bound (input processed in parallel). Document extraction, with long input and short output, already spends much of its time in a regime where it was never starved.

## How Diffusion Helps

Diffusion borrows from image generation: it starts with masked/noisy tokens and refines the whole block in parallel over several denoising passes — "less like a typewriter and more like an editor revising a full draft at once." Each pass does real arithmetic across the whole block, making it compute-bound even at batch size one. Where autoregressive intensity is near 1, comparable diffusion models land in the hundreds.

The authors cite real numbers: Inception Labs' Mercury reported over 1,100 tokens per second on H100s for code generation; Mercury 2 in 2026 reported roughly 1,000 tokens/second on Blackwell at low latency. Google's Gemini Diffusion is in enterprise preview, and open-source LLaDA shows diffusion follows autoregressive-like scaling laws. Mercury 2 is commercially available, and Gemini Diffusion's general availability is expected later in 2026.

## The Fair Comparison

Before declaring a winner, the authors note what autoregressive serving already offers. Speculative decoding (Medusa, EAGLE) uses a small draft model to propose several tokens verified in a single pass, giving roughly two- to four-fold single-stream speedups. Mixture-of-experts models activate only a fraction of weights per token, moving less memory per generation. The real question is whether "diffusion's structural parallelism beats a speculatively decoded model's incremental gain on the workload you actually have." For tight single-stream latency, diffusion's edge is large and durable. For offline batch, neither trick matters much — batching already pushes both into compute-bound territory.

## The Economics

The authors present a core identity: Effective cost per token = node cost per hour ÷ (throughput × utilization). Public APIs are priced per token, concurrency-independent, with no idle penalty. Owned compute is priced per hour, so throughput and utilization are the only levers. Diffusion moves throughput decisively — but only where batching is unavailable.

Concrete pricing: a reserved AWS p5.48xlarge (eight H100s) runs about $33/hour on a one-year savings plan. Against a cheap commodity API (under $1 per million tokens), owned compute loses regardless of architecture. Diffusion's economic win appears in only two situations: when the token you'd otherwise buy is expensive (frontier/reasoning output at $5–$15 per million, where a saturated owned node undercuts the API), or when data sovereignty prevents using external APIs entirely. Most regulated enterprises live in that second case.

## The Bank's Two Workloads

The bank's document operation has two faces that "look alike and behave like opposites":

**Overnight batch** (KYC packets, letters of credit parsed while no one waits) is the easiest possible workload to batch. Continuous batching lets a standard model run at several thousand tokens/second. Diffusion is somewhat faster, but both fit on one box at similar cost. This job is mostly prefill anyway.

**Real-time path**: a relationship manager onboarding a customer needs documents parsed in under a second. These requests arrive one at a time with hard latency budgets — you can't batch them. A large autoregressive model in single-stream decode emits only tens of tokens/second; a few hundred tokens take several seconds. Diffusion returns the same record in well under a second. The cost shows up as node count: to hit subsecond targets with autoregressive, you must keep batches tiny, so each node serves only a handful of concurrent requests. Diffusion clears each request fast enough that one node absorbs far more low-latency traffic, requiring far fewer nodes.

## The Routing Rule

The generalization has two axes: whether work can be batched (offline-tolerant vs. latency-bound) and what each token is worth.

- **Latency-bound, decode-heavy, low-value generation** (code completion, real-time extraction, agentic chatter) is the diffusion sweet spot — batching unavailable, quality gap tolerable, fast owned node beats both overprovisioned autoregressive fleets and expensive APIs.
- **High-value reasoning** (where wrong answers are costly) stays on frontier autoregressive models.
- **Offline batch** of any value density goes to whatever you already run well, because batching has already made it efficient.

## Diffusion's Real Constraints

- **Quality tradeoff**: diffusion trades accuracy for speed, landing around 85–95% of strong autoregressive baselines — competitive on structured output but trailing by 5–15% on hard reasoning. That's fine for field extraction, not fine for credit decisions.
- **Being compute-bound is itself a cost**: diffusion earns its high intensity partly by doing more total work per useful token. The metric that matters is "tokens per dollar at an acceptable quality bar and never utilization on its own."
- **The baseline is moving**: speculative decoding, better schedulers, and MoE models keep narrowing the gap.
- **Tooling is early**: open-source diffusion serving in 2026 sits roughly where open-source autoregressive serving was in early 2024 — functional and improving fast but lacking mature inference stacks like vLLM or TensorRT-LLM.

## Conclusion

The authors argue the hangover is not that enterprises bought the wrong hardware. Many bought it for sovereignty, data control, and avoiding lock-in — reasons unrelated to token economics that won't go away. They expected it to behave like a public cloud but ran it at a concurrency that cloud economics depend on and that their most valuable internal workloads can never reach. Text diffusion is "not a way to beat the API, nor a blanket upgrade for everything an enterprise runs." It's a precise tool for the latency-bound, decode-heavy, sovereignty-constrained work where batching is impossible. "For the copilots, the real-time checks, and the agentic steps that have to answer now," it turns an idle node into a saturated asset on a fraction of the boxes the alternative would need.
