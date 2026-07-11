---
url: https://zenodo.org/records/21115798
title: "What Broke & Why: Practical Lessons from Reinforcement Learning Post-Training"
author: Luv Verma
date_fetched: 2026-07-11
date_published: 2026-07-01
---

# What Broke & Why: Practical Lessons from Reinforcement Learning Post-Training

**Author:** Luv Verma (ORCID: 0000-0002-3835-7836)
**Publication Date:** July 1, 2026
**Publisher:** Zenodo
**DOI:** 10.5281/zenodo.21115798
**License:** Creative Commons Attribution 4.0 International

## Description

A practitioner's guide to reinforcement learning post-training, targeting setups with "one to eight card, H100-80GB" GPU budgets. The book takes a failure-first perspective where each lesson comes from a training run that broke in a specific way, and every reported number is grounded in actual evaluation logs or artifacts.

## Structure

Three layers:

1. **The Journey** — a sequence of programs run (math, search, entropy, mixture-of-experts, SWE-bench, distillation), each failing in a way that motivated the next step.
2. **The Science** — what the runs revealed about how RL alters a model.
3. **The Reference** — a symptom-indexed debugging field guide, including a catalog of failure modes and fixes.

## Topics Covered

- Verifiable-reward RL
- Reward design and reward hacking
- Entropy collapse and training stability (including Clip-Cov and GSPO)
- Correctness-gated rewards
- MoE routing under RL
- Evaluation discipline and small-eval noise
- The rollout engine
- Disaggregated inference

## Keywords

RLHF, Reinforcement Learning, post-training, GRPO, GSPO, reward modeling, LoRA, entropy collapse
