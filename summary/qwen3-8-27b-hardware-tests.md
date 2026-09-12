---
url: https://www.hardware-corner.net/qwen3-8-27b-hardware-tests/
title: "Qwen3.8 27B Hardware Tests"
author: Hardware Corner
date_fetched: 2026-08-21
date_published: n.d.
site: hardware-corner.net
tags: [qwen, local-inference, llama.cpp, gpu, hardware, vram, benchmark]
topics:
  - local-and-open-source-inference
---

# Qwen3.8 27B Hardware Tests

A llama.cpp benchmark of Qwen3.8 27B (Q4_K Small, 16.68 GiB, 27.32B params) run
across a spread of consumer GPUs and unified-memory systems to answer one
question: what does it actually take to run the model locally, especially at long
context. The scope is deliberately hardware-only — VRAM usage, context scaling,
prompt-processing speed, and token generation speed — with no attempt to judge
intelligence, coding quality, or reasoning.

The headline finding is that VRAM, not compute, is the first limit. Measured
usage rises predictably with context: 18 GB at 4k, 22 GB at 64k, 26 GB at 128k,
and 34 GB at 256k. That makes a 24 GB card practical up to 64k and a 32 GB card
sufficient for 128k, while 256k sits beyond any consumer GeForce card.

On the cards tested, the RTX 3090 remains the value pick (~40 tok/s at 4k, ~34 at
64k), the RTX 4090 adds throughput but no extra VRAM, and the RTX 5090 is the
only single-GPU route to 128k — though its generation rate collapses from 74.83
tok/s at 4k to 22.79 at 128k. The dual RTX 5060 Ti (~32 GB aggregate) is framed
as capacity-per-dollar rather than speed; the M5 Max and NVIDIA GB10 are
high-capacity, low-throughput alternatives for very long contexts.

The two comparisons matter most. Against Qwen3.6 27B, Qwen3.8 is not a speed
upgrade — at 64k on the RTX 5090 it generates 26.22 tok/s versus 63.66, a 2.4×
regression. Against Meta's Muse Glimmer 30B, Qwen3.8 is slower at every context
length and less memory-efficient at long context. The article concludes that
Qwen3.8 behaves "much more like Qwen3.6" than like Muse Glimmer, and that long
context carries a real performance cost on any card.
