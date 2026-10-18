---
url: https://github.com/mmaaz-git/agentic-pbt
title: "Agentic Property-Based Testing (agentic-pbt)"
author: Muhammad Maaz, Tom DeVoe, Wren Hatfield-Dodds, Nicholas Carlini
date_fetched: 2026-10-18
date_published: 2025-10
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

The open-source code artifact for the Anthropic paper on agentic property-based testing (arXiv:2510.09907). The whole agent is a single Claude Code slash command (`hypo.md`) that mines properties a codebase *claims* to have — invariants, round-trips, inverse operations, metamorphic relations — writes Hypothesis tests for them, triages failures against a rubric, and emits standardised bug reports. Everything else is infrastructure: `run.py` (507 lines, stdlib only) is a parallel harness that ran the agent over 100+ Python packages at module level, one sandboxed worker per module with an allowlisted Claude Code permission set, resumable via `call_mappings.jsonl`; `scoring.py` re-scores the resulting bug reports through the Claude API using a 15-point rubric (obviousness, input reasonableness, maintainer defensibility). The repo also ships the paper's full run data under `paper/` — logs, aux files, and the five bug reports actually filed with maintainers.

The design philosophy is "the prompt is the product": nearly all of the intelligence lives in an English document, while the Python wrapper contributes only sandboxing, parallelism, and resumability. Triage is as engineered as discovery — the agent is told to prefer *sound but incomplete* strategies, to only test properties the code explicitly claims, and to classify findings by bug type and severity before writing anything down. A second Claude Code command (`utils/check-issue-exists.md`) de-duplicates against existing GitHub issues via the `gh` CLI, with explicit rate-limit survival instructions.
