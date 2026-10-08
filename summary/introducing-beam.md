---
url: https://reflection.ai/blog/introducing-beam
title: "Introducing Beam: Reflection's 501B Open-Weight Model"
author: Reflection AI
date_fetched: 2026-10-08
date_published: 2026-10-05
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Reflection AI announces Beam, its first open-weight model: a sparse Mixture-of-Experts model with 501B total parameters and 23B active, built for coding, reasoning, and agentic workloads, with weights promised under Apache 2.0 later in October 2026.

The headline claim is efficiency at inference time: Beam is competitive with GLM 5.2 on coding and agentic benchmarks while using 3–4× less inference compute, and approaches the much larger Qwen 3.8-Max — though frontier open models like Kimi K3 still lead on raw capability. The engineering story behind that is a very large high-compute RL run (100M+ rollouts on 10.5K NVIDIA GB300 GPUs over four weeks, 1.3B sandboxes, 256K-token rollouts) using fully asynchronous policy gradients that stay stable even when learning from interactions generated more than a day earlier. A controllable length penalty and a user-facing "reasoning effort" parameter let operators trade tokens for performance.

The post is equally detailed on infrastructure: 110K concurrent rollouts, 170K concurrent sandboxes across 20 clusters, two clouds and four regions, 12-second median weight distribution to the inference fleet, 71 inference incidents absorbed without killing the training job, and 92.3% goodput on a four-week 6,144-GPU pretraining run with nine semi-automatic rewinds. Data curation gets the same treatment — 23.8T pretraining tokens, ~95% of raw web tokens eliminated, and a claim that conventional filters would have missed 1.8T high-quality tokens they retain.

Safety and alignment ran as a second model from the same pretrained checkpoint, merged via multi-teacher on-policy distillation (MOPD), with a three-tier principle structure and adversarially iterated safety data; the team reports reward forecasts (r = 0.79) that beat the Best-of-N ceiling as a predictor of RL gains. The takeaway the post wants readers to have: "more intelligence per token" — a Western open-weight workhorse for enterprise coding and agentic workloads, released with the full stack to run, evaluate, and fine-tune it.
