---
url: https://arxiv.org/html/2607.05188v1
title: Latent Programming Horizons in Coding Agents
author: André Silva, Han Tu, Martin Monperrus
date_fetched: 2026-07-18
date_published: 2026-07
---

# Latent Programming Horizons in Coding Agents

André Silva, Han Tu, Martin Monperrus — KTH Royal Institute of Technology, Stockholm, Sweden

## Abstract

The authors investigate what language models underlying coding agents internally represent about the program being edited. Using logistic-regression probes on hidden states across agentic trajectories, they decode whether code "parses, passes its test suite, reduces the number of failing tests, and introduces regressions," achieving AUC up to 0.83 for correctness across two models and two benchmarks. Their second finding is that representations "run ahead of the agent's own edits" — probes predict future edit outcomes above chance up to roughly 25 steps in advance, which they term the agent's "latent programming horizon."

## 1. Introduction

Coding agents iteratively edit code over many steps. Prior work shows transformers develop linear representations of game states and synthetic program states. Prior correctness probing assumed single-step generation with full programs in context, not iterative editing of partially observed real codebases. The authors collect agentic trajectories, extract residual streams, and train linear probes. Two main findings: (1) agents encode current program properties, and (2) agents anticipate future programs up to ~25 steps ahead. Four contributions: a novel experimental protocol, evidence that linear probes decode four program properties, demonstration that probes transfer across benchmarks, and evidence of the latent programming horizon.

## 2. Latent Program Representations

The residual stream carries information from input through output. Three definitions: "latent program representation" (what is encoded about a program in the residual stream, independent of surface tokens), "latent program space" (the manifold of hidden-state space along which representations are encoded), and "latent programming horizon" (the extent to which future edits are already encoded in the current residual stream, measured by predicting program properties k steps ahead). Next-token prediction and post-training signals (execution traces, compiler/test feedback, RL) should contribute to semantically rich latent representations.

## 3. Methods

**3.1 Overview:** Linear probes classify program properties from hidden states.

**3.2 Formal Setup:** Agentic trajectories consist of S interaction steps. Edit events occur when tool calls modify code. Program properties are binary labels assigned per edit event.

**3.3 Program Properties:** Well-formedness (parses/compiles), Full Correctness (passes test suite), Partial Correctness (fewer failing tests than at start), Regression (previously passing tests now fail).

**3.4 Probing Program Properties:** Logistic regression classifiers trained on hidden states from layers {1, 11, 21, 31, 40}, with shuffled-label controls.

**3.5 Programming Horizon:** Probes trained to predict labels k steps ahead (k from 0 to 50), excluding final k_max steps of each trajectory.

**3.6 Dataset:** Two agents (Qwen3.6-35B-A3B, Laguna-XS.2, both d=2048) running mini-swe-agent v2.2.8 on SWE-Bench-Verified (500 tasks) and SWE-Bench-Pro (731 tasks). Up to 10 trajectories per task, 22,714 total trajectories, 79,480 total edits, 22.4M hidden-state vectors collected every 5 tokens. Median steps = 52, median edits = 2.

## 4. Results

**4.1 Coding agents encode the current program:** Every probe decodes its property above chance. Full Correctness reaches AUC up to 0.83, Partial Correctness up to 0.84. Well-formedness on Verified collapses near chance due to label imbalance (92%+ positive). Inverted-U pattern across layers: weakest at layer 1, peaks in intermediate layers, slight drop at final layer. Qwen3.6 encodes more strongly than Laguna (~0.10 AUC gap). Probes transfer across datasets with small drops (0.04–0.09 AUC).

**4.2 Coding agents have a long-term programming horizon:** Predictive signal decays smoothly with horizon. At the edit, best-layer probes recover Full Correctness at AUC ~0.77–0.82. Prediction remains above 0.5 for at least 25 steps, plateauing above chance out to k=50. This is "the first evidence of long-term horizon by coding agents."

## 5. Related Work

Six areas: probing internal representations (Alain & Bengio, Nanda et al., Hewitt & Liang), predicting future states (Jenner et al. on chess, Zhang et al. on CoT correctness), code representations in LLMs (Jin & Rinard on Karel programs, Hernández López et al. on ASTs), correctness probing (Ribeiro et al., Bui et al., Vu et al. — all on single-step generation), latent program spaces (Neural Programmer-Interpreter, continuous program spaces), and coding agents (SWE-agent, OpenHands, CodeAct, AutoCodeRover).

## 6. Limitations

(1) Decodability is not causality — probes don't establish causal use of representations; (2) label imbalance affects Well-formedness probes; (3) scope limited to two open-weight models under one agent scaffold on two benchmarks.

## 7. Conclusion

Coding agents encode program properties in residual streams. Linear probes decode compilation, test pass/fail, regression, and partial correctness up to 0.83 AUC. Representations ground predictions of future edits up to 25 steps ahead. Results hold across models, benchmarks, and transfer between datasets. The authors state these "open novel research directions towards monitoring and steering coding agents from within the latent space."

**Acknowledgments:** Supported by the Wallenberg AI, Autonomous Systems and Software Program (WASP), computational resources on Berzelius system, and compute support from Modal.

**References:** 58 references spanning mechanistic interpretability, code generation, and agentic systems.
