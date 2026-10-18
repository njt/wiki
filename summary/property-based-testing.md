---
url: https://www.anthropic.com/research/property-based-testing
title: "Finding bugs with property-based testing"
author: Muhammad Maaz, Liam DeVoe, Zac Hatfield-Dodds, Nicholas Carlini (Anthropic)
date_fetched: 2026-10-18
date_published: 2026 (NeurIPS 2025 Deep Learning for Code Workshop paper)
topics:
  - guardrails-and-feedback-loops
  - software-engineering-craft
---

Anthropic's Frontier Red Team built an agent — a custom Claude Code command — that autonomously writes property-based tests for existing code and found hundreds of candidate bugs in popular Python packages including NumPy, SciPy, and Pandas, several already patched upstream.

The core move is pairing LLM property inference with property-based testing (PBT). Instead of example-based unit tests with hand-picked inputs, the developer (or the agent) states a general invariant — e.g. "JSON deserialization is the inverse of serialization," or "Wald-distribution samples are positive" — plus an input domain, and a framework (Hypothesis) fuzzes for counterexamples. The agent derives properties from type annotations, docstrings, function names, comments, and existing tests, uses a to-do list for long-range reasoning, and runs a self-reflection loop to cut false alarms (one example: catching that it had wrapped its own test in try/except). Newer models (Opus 4.1, Sonnet 4.5) showed notably better self-reflection than Sonnet 4.

Evaluation ran in two phases. In the first, Opus 4.1 produced 984 bug reports across 100+ popular packages; a manual sample found 56% valid, 32% valid-and-reportable. A 15-point rubric scored by Opus 4.1 made triage dramatically more effective: top-scoring reports were 86% valid, 81% reportable. The second phase added an evaluation agent and paid expert human reviewers. Responsible disclosure got heavy treatment — three human reviewers per bug, ~1 hour each, authors reviewing again before filing — with invalid reports published too.

Real finds include `numpy.random.wald` returning negative values (catastrophic cancellation; their fix had ~10 orders of magnitude lower relative error), a dictionary-slicing iterator bug in AWS Powertools, and a missing parenthesis in Hugging Face tokenizers. The dateutil rejection shows the limit: when code has subtle implicit semantics, only maintainers can say what the correct property is. The authors see agentic PBT as a complement to human-written testing, and a natural stepping stone toward automatic patch generation.
