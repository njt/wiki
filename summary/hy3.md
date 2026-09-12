---
url: https://huggingface.co/tencent/Hy3
title: "Hy3 — Tencent's Large MoE Language Model"
author: Tencent Hy Team
date_fetched: 2026-07-11
date_published: 2026
topics:
  - ai-research-and-models
---

Hy3 is Tencent's large Mixture-of-Experts model: 295B total parameters with 21B active and 3.8B in a multi-token prediction (MTP) layer. It supports a 256K context window, uses 192 experts (top-8 activated), and is Apache 2.0 licensed.

The team claims Hy3 outperforms similar-size models and rivals open-source flagship models with 2–5× more parameters. A blind expert evaluation scored it 2.67/4, ahead of GLM-5.1 at 2.51/4, with the largest lead in frontend development, data & storage, and CI/CD tasks.

Three product-improvement areas are highlighted: tool-call stability (≤4% accuracy variance across CodeBuddy, Cline, and KiloCode scaffoldings on SWE-Bench Verified), hallucination reduction (12.5% → 5.4%), and multi-turn tracking (internal issue rate dropped from 17.4% to 7.9%).

Notable benchmarks include GPQA Diamond 90.4, SWE-bench Verified 78, and HLE 53.2. Deployment is supported via vLLM and SGLang, with an FP8 quantized version and an AngelSlim compression toolkit also available.
