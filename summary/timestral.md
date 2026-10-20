---
url: https://mistral.ai/news/timestral/
title: "Timestral: our first family of time-series foundation models"
author: Mistral AI
date_fetched: 2026-10-20
date_published: 2026-10-20
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Mistral announces Timestral, its first family of time-series foundation models (TSFMs): four sizes (5M, 28M, 123M, 209M parameters) doing probabilistic, multivariate zero-shot forecasting. The 209M flagship claims first place on the field's three public benchmarks — GIFT-Eval, TIME, and fev-bench — using two-thirds the parameters of the previous state of the art, from a single checkpoint with no ensembling.

The pitch rests on the foundation-model argument familiar from LLMs: classical models (ARIMA) learn from a single series; deep-learning models (PatchTST) learn nonlinear patterns but must be retrained per dataset; a TSFM pre-trains on a large diverse corpus and transfers across domains. Every model size claims a spot on the Pareto front of accuracy vs. size — the 28M beats Chronos-2 at ~4× its size, the 209M beats TimesFM-3 and Falcon-2.0 with fewer parameters. Inference runs on CPUs: a one-day-ahead forecast from three weeks of 15-minute history takes 10 ms (5M) to 140 ms (209M) on an 8-thread server CPU, no GPU needed.

Three technical contributions are named ahead of a promised technical report: Decoupled Horizon Forecasting (forecast tokens read the full history instead of masking it away, so every observed value serves as both target and context); covariate-aware attention (roles for target, past-only, and known-future covariates, so a holiday calendar can inform a forecast without leaking the target's future); and synthetic-first training data, with a two-stage schedule (8192-context synthetic pre-training, then real+synthetic post-training at 4× context) optimised with pinball loss over 21 quantiles.

Positioning leans industrial: sensor telemetry, capacity planning, energy, supply chains. The marketing set-piece is an agentic workflow — Timestral flags an anomaly on a pump, an agent compares embeddings against historical failures, sub-agents run root-cause analysis over maintenance records and shift logs, and a diagnosis is drafted. Deployment is via Mistral Forge fine-tuning, on-prem/VPC/air-gapped, under a modified MIT licence, with a GitHub repo now public.
