---
url: https://injuly.in/blog/napkin-inference-cost/index.html
title: "Inference cost at scale with napkin math"
author: injuly.in (unnamed)
date_fetched: 2026-06-21
date_published: 2026-06-14
topics:
  - ai-infrastructure-and-hardware
---

# Inference cost at scale with napkin math

Published June 14, 2026 on injuly.in. Tagged GPU, AI, math.

## Core Argument

With basic knowledge of GPU specs (peak throughput in TFLOPs, memory bandwidth in TB/s) and model architecture, you can estimate the dollar cost per user of serving AI models. The central tension is between GPU memory bandwidth and compute throughput — matching these determines how many concurrent users a chip can efficiently serve.

## GPU Specs

Two critical specs: **peak throughput** (TFLOPs/second) and **memory bandwidth** (TB/second). The article assumes FP-8 quantization.

## Matrix Multiplication Cost

For matrices A(N×d) and B(d×M), the product requires 2NMd memory accesses and 2NMd floating-point operations. With tiling, memory access drops to roughly d(N+M).

## Language Model Architecture

LLMs receive N tokens as d-dimensional vectors, apply attention across layers, and predict the next token autoregressively — feeding outputs back as inputs until a stop token.

## Attention & KV-Cache

Without caching, processing one token for one user with N=200k and d=8192 requires **26 trillion FLOPs** and **1.7 billion memory accesses** — a severe compute-to-memory imbalance. The KV-cache solves this by storing K and V matrices from prior tokens, so each forward pass processes only the newest token.

With KV-cache, the math shifts to ~52.4 million ops and ~26.2 million memory accesses per batch — a ratio of **2×B operations per memory access**.

## Balancing GPU Resources

The B200 has 8 TB/s memory bandwidth and 4500 TFLOP/s compute — it "can crunch bytes **562 times faster** than it can load them." The ideal batch size solves 2B = 562, giving **B = 331 concurrent users** as a theoretical ceiling.

## VRAM Constraints

A 32B model uses 32GB for weights. KV-cache for a 200k context is **210GB** without optimization. Using Grouped-Query-Attention (GQA) cuts the cache ~8× to **~26GB per user**. With 160GB remaining after weights, that's only **~6 concurrent users** at full context length.

## PagedAttention & Real-World Users

By allocating KV-cache incrementally and flushing cold sessions, "you can serve anywhere between **40-60 users per Blackwell chip**." Accounting for user idle time (~80% reading), one chip can handle **~300-800 users** depending on app style.

## Tokens Per Second

"For a single forward pass we move all the model weights + KV-cache from VRAM to registers once." Time breakdown: ~23.75ms moving data, ~0.5ms computing. This yields **~40 tokens/second per user** for 6 users — "beyond most people's reading speed."

## Dollar Cost Per User

- **Ownership:** At $40,000/B200, lifetime cost is $133/user at 300 users.
- **Rental:** At ~$4/hour, that's "$0.013 per user, or `$9.36` per month."

The author calls this "a rather conservative estimate" with headroom for high-duty-cycle workflows like agentic loops.

## Key Quotes

- "Inference engines will cache the K,V pairs for reuse"
- "the compute cores are idle 98% of the time"
- "For non-chat apps, measuring duty-cycles is not optional"

## Caveats

Assumes dense transformer architecture (not Gated Delta-Nets or Gemma 3-style optimizations). Multi-GPU setups require moving beyond napkin math.
