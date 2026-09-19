---
url: https://prismml.com/news/bonsai-2-27b
title: "Introducing Bonsai 2 27B: Near-Lossless Compression in a 9x Smaller Footprint"
author: PrismML
date_fetched: 2026-09-19
date_published: 2026-09
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

PrismML released Ternary Bonsai 2 27B, the second generation of their compressed
model line, built on Qwen3.8 27B rather than the first release's Qwen3.6 base.
The model uses ternary {−1, 0, +1} weights with FP16 group-wise scaling for
1.76 effective bits per weight and a 5.9 GB total footprint — more than 9x
smaller than the full-precision model — with the low-bit representation applied
end to end across the language model. It keeps the series' fixed deployment
profile: 262K-token context, multimodal text-and-image input, Apache 2.0.

The headline claim is capability retention: an aggregate benchmark score of 83.9
across reasoning, math, coding, instruction following, vision, and agentic tool
use, retaining 98.2% of Qwen3.8 27B's full-precision performance — up from ~95%
for the first Bonsai 27B generation. PrismML argues the retention matters most
precisely where degradation compounds: coding agents, tool-use systems,
multimodal workflows, and long-horizon tasks, and positions the model as an
outlier on "intelligence density" against other low-bit alternatives that give
up coding, vision, or agentic capability to become deployable.

Throughput is up to 143 tok/s on an RTX 5090 and 46.8 tok/s on an M5 Max. On an
RTX 4090 the model consumes 0.714 mWh/token, which PrismML says makes it 40%
more energy-efficient than a full-precision 8B model. It runs on NVIDIA GPUs
via CUDA and Apple devices (Mac, iPhone, iPad) via MLX through custom low-bit
kernels; weights are available under Apache 2.0, with full compression and
evaluation detail deferred to a whitepaper.

Strategically, PrismML pitches local models taking on real knowledge work —
coding-agent loops, computer-use workflows, private document analysis, and
hybrid orchestration where local models handle sensitive or high-frequency
tasks and escalate selectively to the cloud — and extends the argument beyond
devices: if capability scales while memory, compute, and power requirements
fall, the deployment envelope expands from personal devices to datacenters.
