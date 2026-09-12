---
url: "https://coles.codes/posts/local-models-mid-2026"
title: "Local models in mid-2026: the engineering that closed the gap"
author: "Matt Coles"
date_fetched: 2026-06-15
date_published: 2026-06-12
source: "coles.codes"
tags:
  - "#local-inference"
  - "#AI-research"
  - "#hardware"
  - "#concept"
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

# Local models in mid-2026: the engineering that closed the gap

Article by Matt Coles (github.com/MattJColes), published June 12, 2026. ~11 minute read.

## Core Thesis

Open-weights models haven't fully matched the frontier, but they've become "close enough" for everyday work — writing, research, and specialized agent tasks. The key driver wasn't bigger hardware but engineering advances that reduced compute and memory per token without sacrificing quality.

## Favorite Models

| Model | Parameters | Architecture | Notes |
|-------|-----------|-------------|-------|
| Qwen 3.6 | 27B dense; 35B MoE | Mixture-of-Experts | Only ~3B active params per token |
| Gemma 4 | Multiple sizes | Dense + MoE variants | Larger sizes "punch well above their weight" |
| GLM-5 | 744B MoE | Mixture-of-Experts | Too much RAM for author's local setup |
| Kimi K2.6 | ~1 trillion params | MoE with 32B active | 384 experts, 8 active + shared expert per token |
| DeepSeek V4 (Flash & Pro) | MoE | 1M-token context | Previewed April 2026 |

Nearly every model mentioned is sparse/MoE, not dense.

## Sparse Attention

Standard self-attention is O(n²). DeepSeek's solution (V3.2 → V4): "lightning indexer" — a cheap FP8 scoring function that selects top-k earlier tokens for each query. A small sliding window maintains full-resolution local coherence. The indexer runs on a separate CUDA stream, so latency is hidden behind already-occurring work.

V4-Pro reportedly needs ~75% fewer per-token inference FLOPs and ~90% less KV cache than V3.2 at million-token context.

## Mixture-of-Experts

MoE replaces one big dense feed-forward network with many smaller "expert" networks plus a router per token.

- Kimi K2.6: 384 experts, activates 8 plus 1 shared per token
- GLM-5: Activates ~40B of its 744B total params

Key tension: All experts must be held in memory, but only a few are touched per token. MoE is "cheap on compute and bandwidth but very heavy on capacity" — making unified-memory machines a surprisingly good fit.

## The KV Cache Problem

For long-context and reasoning models (which may emit 20K+ tokens of chain-of-thought), the KV cache often dominates memory cost over the weights themselves.

Mitigation strategies:
1. **Multi-head Latent Attention (DeepSeek):** Compresses KV cache into low-rank latent representation, ~90% footprint cut
2. **Lower-precision cache storage:** FP8 → FP4 halves/quarters memory with minor accuracy loss

Together, compressed attention + compressed quantized cache "moves the long-context memory wall a long way out."

## Multi-Token Prediction

Normal autoregressive generation is memory-bandwidth-bound. A small "drafter" guesses several tokens ahead; the full model verifies all guesses in a single parallel pass.

- "Lossless" — big model checks every token, output quality is identical
- DeepSeek reported ~85-90% acceptance rate for 2nd predicted token, ~1.8x throughput gain
- Gemma 4 ships dedicated small drafter models sharing embeddings and KV cache
- Caveat: gains are workload-dependent; high-entropy output sees more rejected drafts

## Four-Bit Quantisation

FP4 (NVFP4 and MXFP4 formats) moving from research to shipping:
- OpenAI released gpt-oss natively in MXFP4
- Nvidia Blackwell does FP4 in hardware
- Qwen 3.6 27B drops from ~17GB at 4-bit-ish quant to ~14GB in NVFP4
- Quantization-aware training recovers most accuracy from naive rounding

Drawback: "does cost you accuracy on small or sensitive models" where small block sizes interact poorly with outliers. For larger models, "a sensible default rather than a compromise."

## The Memory Supply Crunch

Hardware prices surged due to AI demand:
- Memory makers shifted capacity toward datacenter HBM
- PC DRAM passed 100% increase; 1TB SSDs roughly doubled in price
- SK Hynix sold out next year's capacity
- Relief not expected before late 2027

"The models are finally good enough to run at home right as the box to run them on got expensive."

## Hardware Recommendations

| Option | Details |
|--------|---------|
| Apple Silicon Mac Studio | Unified memory, fits MoE well |
| AMD Strix Halo mini-PC | Up to 128GB shared CPU/GPU |
| Used RTX 3090 | "Budget pick" |
| RTX 5090 | "Fast one if you have the $$$" |
| DGX Spark (current gen) | Author owns one; bandwidth/cooling concerns |
| RTX Spark | Announced May 2026; Grace CPU + Blackwell GPU; up to 128GB unified memory; ships fall 2026 |

Author is watching RTX Spark most closely. Unified memory advantage for MoE: lots of capacity needed (all experts in memory) but only moderate bandwidth (activating few per query).

## Where the Gap to Closed Models Sits

Epoch's measurement: Best open weights ~4 months behind closed frontier — slightly wider than the ~3-month average of prior years.

Artificial Analysis Intelligence Index (June 2026):
| Model | Score | Comparison |
|-------|-------|-----------|
| Kimi K2.6 | 54 | vs GPT-5.5 at 60, Claude Opus 4.7 at 57 |
| DeepSeek V4 Pro | ~tied | Level with Sonnet 4.6 |
| Qwen 27B / Gemma 31B | Tier below | Small dense models for single GPU |

Coding & agentic work: open models within a few points of closed. Hard reasoning & novel math: closed frontier still ahead.

Caveats: "Every model benchmaxes these days" — public scores run high. Claude Fable 5 released as author was writing; if early numbers hold, the 4-month gap figure is already stale on the optimistic side.

Bottom line: "a good-enough model in a good harness already covers their day-to-day."

## Why Run Your Own

- **Learning:** "An afternoon spent working out why the same model does fourteen tokens a second on one machine and forty on another" teaches more than a month of reading
- **Flexibility:** Pull a model on release day, quantize it, fine-tune on personal data, keep sensitive workloads on controlled hardware

Key insight: Sparse attention, MoE routing, latent KV compression, multi-token prediction, and 4-bit quantization are all published papers and merged commits, not trade secrets. "The models being good is nice, but it's the methods staying out in the open that gives the rest of us options."
