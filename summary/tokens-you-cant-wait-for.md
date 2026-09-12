---
url: https://www.oreilly.com/radar/the-tokens-you-cant-wait-for/
title: "The Tokens You Can't Wait For: Text diffusion, the GPU hangover, and the one place parallel generation actually pays off"
author: Shreshta Shyamsundar and Anmol Jain
date_fetched: 2026-07-21
date_published: 2026-07-20
topics:
  - ai-infrastructure-and-hardware
---

This O'Reilly Radar piece examines the mismatch between how standard autoregressive LLMs generate text and how enterprises need to use them — and makes the case that text diffusion models solve a specific, expensive subset of that mismatch.

The core problem: autoregressive models generate one token at a time, streaming model weights from GPU memory through compute units for every single token. At batch size one (a single real-time request), arithmetic intensity hovers near 1, while GPUs are built for intensities in the hundreds. Batching fixes this — read weights once, compute hundreds of requests simultaneously — but low-latency workloads (code completion, real-time document parsing, onboarding checks) can't wait to fill a batch. That leaves expensive hardware severely underutilized.

Text diffusion borrows from image generation: it starts with masked tokens and refines the entire output block in parallel over several denoising passes, making it compute-bound even at batch size one. Where autoregressive intensity is near 1, comparable diffusion models land in the hundreds. Inception Labs' Mercury reported over 1,100 tokens/second on H100s; Mercury 2 reached roughly 1,000 tokens/second on Blackwell. Google's Gemini Diffusion is in enterprise preview, and open-source LLaDA shows diffusion follows autoregressive-like scaling laws.

The authors walk through the economics with a concrete case: a Singapore bank bought eight H100s for sovereign AI and now faces utilization problems. Their document-processing work has two faces. Overnight batch processing (KYC packets, letters of credit) is trivially batchable — standard models already run efficiently there. But real-time document parsing for customer onboarding arrives one request at a time with sub-second latency budgets. Autoregressive models at single-stream decode emit only tens of tokens/second; diffusion clears the same work in well under a second, letting one node absorb far more concurrent low-latency traffic.

The routing rule the authors propose has two axes: whether work can be batched, and what each token is worth. Latency-bound, decode-heavy, low-value generation (code completion, real-time extraction, agentic chatter) is diffusion's sweet spot. High-value reasoning stays on frontier autoregressive models. Offline batch of any value density goes to whatever already runs well.

Diffusion's constraints are real: it trades accuracy for speed (85–95% of strong autoregressive baselines), its compute-bound nature means it does more total work per useful token, the autoregressive baseline keeps improving with speculative decoding and MoE, and diffusion serving tooling is several years behind — roughly where open-source autoregressive serving was in early 2024.

The conclusion: diffusion isn't a blanket upgrade. It's a precise tool for the latency-bound, decode-heavy, sovereignty-constrained work where batching is impossible — turning idle nodes into saturated assets on a fraction of the boxes the alternative would need.
