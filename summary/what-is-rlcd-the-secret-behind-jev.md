---
url: https://di-zhang-llm.github.io/blog/what-is-rlcd-the-secret-behind-jev/
title: "What Is RLCD? The Secret Behind Jev"
author: Di Zhang
date_fetched: 2026-09-25
date_published: 2026
topics:
  - ai-research-and-models
  - coding-agents-and-frameworks
---

Di Zhang's technical deconstruction of TypeSafe's Jev argues that Jev is not a new kind of language model but a reward model promoted into the product itself. The lineage runs from scalar reward models (whose absolute numbers never really meant anything) through LLaMA-Berry's pairwise preference model PPRM (Bradley–Terry), to Plackett–Luce as the multiway generalization, with RLCD adding calibration on top via proper scoring rules like Brier.

The architecture is standard: a decision head computing utilities as scaled inner products between a state+question query vector and candidate key vectors (rank 512), softmaxed into a calibrated distribution. Jev's three primitives — `Noul` (binary), `Choice` (K-way), `Score` (ordered levels) — are three schemas over the same Plackett–Luce object. The "parallel sampler" that TypeSafe markets as a new architecture is, on Zhang's reading, just sequence packing plus a tree attention mask plus one shared decision head: established throughput engineering, no new sampler, and conspicuously no ablation in the launch post.

Zhang also rebukes TypeSafe's taxonomy: RLHF/RLVR/RLCD are not three parallel reward *sources* — RLCD is defined by its output contract (typed, calibrated probability distributions), and can be trained from any of those sources. The piece closes with five falsifiable predictions (pairwise/multiway consistency, candidate-set sensitivity, empirical calibration, order symmetry) that turn the interpretation into a testable model of Jev's behavior.
