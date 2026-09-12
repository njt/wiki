---
title: "Optimise Anything"
url: https://gepa-ai.github.io/gepa/blog/2026/02/18/introducing-optimize-anything/
date_fetched: 2026-05-14
section: "Random"
topics:
  - agent-architecture
---

# optimize_anything: A Universal API for Optimizing any Text Parameter

Declarative API that optimizes any text-based artifact -- code, prompts, agent architectures, configurations -- by combining LLM reasoning with Actionable Side Information (ASI) and Pareto-efficient search. Extends GEPA (Genetic-Pareto) beyond prompts to any measurable text artifact.

Three optimization modes:
1. Single-Task Search: Solve one hard problem (e.g., circle packing)
2. Multi-Task Search: Optimize across related problems with cross-transfer (e.g., CUDA kernels)
3. Generalization: Build skills transferring to unseen examples (e.g., prompts, agent architectures)

Two core mechanisms:
- Actionable Side Information (ASI): Diagnostic feedback as first-class API concept. Evaluators return structured diagnostics (error messages, execution traces, rendered images) instead of collapsing to single scalar.
- Pareto-Efficient Search: Maintains frontier of candidates excelling at different metrics, preventing averaging from hiding strengths.

Results across eight domains:
1. Coding Agent Skills: near-perfect pass rates, 47% faster resolution
2. Cloud Algorithms: 40.2% cost savings
3. ARC-AGI: 10-line stub evolved to 300+ lines, 32.5% -> 89.5% accuracy
4. Math: gpt-4-mini AIME 46.67% -> 60.00% via prompt refinement
5. CUDA Kernels: 87% match or beat baseline, 25% achieve 20%+ speedups
6. Circle Packing: outperformed AlphaEvolve, ShinkaEvolve, OpenEvolve
7. Blackbox Optimization: matched Optuna on 56-problem benchmark
8. 3D Unicorn: creative geometry from natural language

"If it can be serialized to a string and its quality measured, an LLM can reason about it and propose improvements."

Backend-agnostic, extensible, community-contributed strategies and evaluators.
