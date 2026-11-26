---
url: https://chszlab.github.io/AgenticAlgorithmEngineering/
title: "Agentic Algorithm Engineering (AAE)"
author: Algorithm Engineering Group, Heidelberg University (Bader, Chhabra, Großmann, Henzinger, Noe, Schulz)
date_fetched: 2026-11-26
date_published: 2026 (paper to appear, DIMACS Implementation Challenge Workshop, arXiv:2609.07204)
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

AAE is a skill for Claude Code and Codex (installable as a Claude Code plugin) that hands the classic Algorithm Engineering cycle — design, analysis, implementation, experiment — to an autonomous agent. During setup the human agrees with the agent on the benchmark, the metric, the correctness assertions, and the budget; the agent then runs on its own under a strict loop: a falsifiable hypothesis before every experiment, correctness checked after every change, keep-or-revert against the benchmark, one reviewable Git patch per accepted change, and a PDF report of the session.

The project page leads with results, not promises: VieCut 8.9x faster with every minimum cut unchanged, KaHIP `kaffpa` 3.9x faster with partition quality unchanged, Hochbaum's pseudoflow 3.0x faster with flow values verified on 164 instances, FREIGHT 1.82x faster with bit-identical output, ParHIP 1.61x faster with byte-identical partitions. Every accepted change is checked against the original output before it is kept.

The page positions AAE against autoresearch systems like AlphaEvolve, OpenEvolve and ShinkaEvolve: those operate on a single training script or code blocks scored by an evaluator with evolutionary populations, while AAE works on any existing code base and language, driven by one agent following the algorithm-engineering cycle, with correctness defined by assertions agreed with the human rather than an evaluator score.

Sessions share what they learn: a shared knowledge base collects optimisation techniques and experiment results, and agents read it before starting. The work is backed by a paper (preprint arXiv:2609.07204) and a Zenodo DOI for the software.
