---
url: https://prismml.com/news/bonsai-27b
title: "Announcing Bonsai 27B — The First 27B-Class Model to Run on a Phone"
author: PrismML
date_fetched: 2026-07-18
date_published: 2026-07-14
topics:
  - local-and-open-source-inference
---

PrismML announced Bonsai 27B, a multimodal model built on Qwen3.6 27B that they
claim is the first 27B-class model capable of running locally on a phone. Two
variants are released: a ternary-weight version (5.9 GB, 1.71 bpw) targeting
laptop-class quality, and a 1-bit version (3.9 GB, 1.125 bpw) designed to fit
within an iPhone 17 Pro's ~6 GB usable memory budget. Both apply low-bit
representation across the entire network — embeddings, attention, MLPs, and the
LM head — with no higher-precision fallback layers.

The 1-bit variant retains roughly 90% of the full-precision Qwen3.6 baseline
across 15 benchmarks, while the ternary variant retains ~95%. Reasoning,
coding, tool-calling, and knowledge benchmarks all degrade but stay usable. The
largest drop is in agentic/tool-calling and instruction-following, particularly
on the 1-bit model (66% and 65.8% vs. the 80% and 78.4% baselines).

Performance is 163 tok/s (1-bit) on an RTX 5090 and 87 tok/s on an M5 Max. The
model supports 262K-token context, speculative decoding, and a compact 4-bit
vision tower for multimodal input. It runs natively on Apple Silicon via MLX
and on NVIDIA GPUs via CUDA, with custom low-bit kernels for a hybrid-attention
architecture.

PrismML frames local execution as transformative for agentic workloads — zero
marginal cost for long reasoning loops and data never leaving the device — and
proposes hybrid cloud/local architectures where privacy-sensitive steps run
locally and frontier-difficulty steps run in the cloud.

Released under Apache 2.0. PrismML emerged from Caltech researchers backed by
Khosla Ventures, Cerberus, Google, and Samsung. Models are on Hugging Face
(`prism-ml/bonsai-27b`) with an API available via Together.ai and an iOS app
(Locally AI).
