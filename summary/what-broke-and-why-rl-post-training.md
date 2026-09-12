---
url: https://zenodo.org/records/21115798
title: "What Broke & Why: Practical Lessons from Reinforcement Learning Post-Training"
author: Luv Verma
date_fetched: 2026-07-11
date_published: 2026-07-01
topics:
  - ai-research-and-models
---

A practitioner's guide to RL post-training for LLMs, written from a failure-first perspective. Each lesson comes from a real training run that broke in a specific way, with every reported number grounded in actual evaluation logs. The target setup is modest: one to eight H100-80GB GPUs.

The book is organized in three layers. **The Journey** walks through a sequence of training programs — math, search, entropy, mixture-of-experts, SWE-bench, distillation — each failing in a way that motivated the next step. **The Science** distills what the runs revealed about how reinforcement learning alters a model. **The Reference** is a symptom-indexed debugging field guide with a catalog of failure modes and their fixes.

Key topics include verifiable-reward RL, reward design and reward hacking, entropy collapse and training stability (covering Clip-Cov and GSPO), correctness-gated rewards, MoE routing under RL, evaluation discipline and small-eval noise, the rollout engine, and disaggregated inference.

The book's practical stance and grounding in concrete failures distinguish it from more theoretical RLHF literature. It is published on Zenodo under a CC-BY license.
