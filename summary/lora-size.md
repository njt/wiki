---
url: https://engram.com/blog/lora-size
title: "Predicting LoRA size for any dataset"
author: Engram
date_fetched: 2026-11-26
date_published: 2026 (not stated precisely)
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Engram develops a predictive theory of LoRA adapter capacity: how many bits a rank-r adapter can learn, as a function of rank, derived from Fisher information and random matrix theory (Marchenko–Pastur), and validated empirically by training on synthetic datasets of exactly known bit-size generated via reversed arithmetic coding ("neural linguistic steganography") under the base model.

Key findings: unlike full fine-tuning, LoRA's capacity per parameter is *not* constant — it is ~1.3–1.4 bits per parameter at low rank and ~0.48 bpp at full rank for an 800M Qwen3.5, with capacity growing roughly linearly at low rank and plateauing as rank rises. Adapting multiple weight matrices jointly is super-additive (total capacity rises from 1.5 to 3.3 bpp across layers). Perhaps most striking: models forget roughly as much as they learn — a model gaining 10 KB of new information sheds ~12 KB of old knowledge.

The practical payoff is a sizing recipe: measure a dataset's information content as its negative log-likelihood (code length) under the base model — using 2M-token YaRN-extended windows — then read the required rank off a synthetic capacity curve. On real datasets (internal emails, law-firm workspaces, SWE-bench Django, GSM8K, enwik8, etc.) the predicted rank comes within ~5% of the smallest rank that stores 95% of the dataset. This lets Engram mint minimally-sized PEFT adapters on the fly — the systems motivation being that smaller LoRAs load faster, generate faster, and store cheaper.

The appendix carries the derivation: second-order log-likelihood approximation, Kronecker-factorized Fisher, truncated-SVD optimality, whitened gradients as iid Gaussians, and a closed-form capacity curve C(r) via the Marchenko–Pastur upper quantile. Also modelled: the log-logistic fit of loss-vs-steps for extrapolating to infinite-training capacity.
