---
url: https://huggingface.co/OlynkAI/piston-v0.1
title: "Piston v0.1"
author: Chennapragada Sathvik (Sasache) and OlynkAI
date_fetched: 2026-11-06
date_published: 2026
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

Piston v0.1 is a small open-weight (Apache-2.0) causal model for time series from OlynkAI. Given a panel of up to 9 variables over 128+ steps and a what-if question (set variable `treatment` to x1 instead of x0 at step t; how much does `target` change k steps later, k=1–3?), it returns an effect with a 90% range, a `p_answerable` probability that the data can answer the question at all, and reachability. It reads data in context — no per-dataset fitting.

Architecture is a ~4M-parameter axial transformer as a 3-seed ensemble (12M total), trained only on synthetic worlds. It refuses to answer when `p_answerable < 0.5`; on the frozen benchmark (4,592 questions in 998 unseen synthetic worlds from 23 world families, with exact true effects and answerability labels — 32% of questions are not identifiable from the panel), baselines answer everything, including the unanswerable third.

Without a causal graph, Piston scores T1 (risk–coverage AUC) 0.280 vs 0.387 for the strongest baseline (CCPFN), 27.6% lower, with 94% interval coverage and 35% confident-wrong — a rate the authors openly say misses their own target of half the best baseline. With a known graph, exact identification + Piston reaches T1 0.112, 37% better than the strongest baseline; the learned refusal is weaker than exact identification. Limitations are candid: synthetic-only training and benchmark (a home advantage over CCPFN), no real-data evaluation, no transfer to contact physics, refusal doesn't detect unfamiliar data, and scope capped at 9 variables, lags 1–3, single interventions. Baseline intervals had to be widened to reach 90% coverage (CCPFN ×6.99, Chronos-2 ×20.7, TimesFM 3 ×55.6); Piston needed no widening.
