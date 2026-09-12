---
url: https://github.com/capitalone/vulnhunter
title: "VulnHunter"
author: Capital One
date_fetched: 2026-07-18
date_published: 2025
topics:
  - security-and-sandboxing
---

Capital One's open-source agentic AI security tool for source code vulnerability hunting. Built as a suite of Claude Code skills (pure prompt engineering) plus Python helper packages that form a closed-loop Hunt → Fix → Verify pipeline.

The hunt skill uses a strict orchestrator-worker pattern: Phase 1 builds an input inventory, threat model, and subgraph partitions via union-find on a two-level call graph. Phase 2 dispatches trace agents — three per partition (specialized by vulnerability class: injection, navigation/auth, logic/crypto) plus a sink-driven auditor — in waves of up to 6 to avoid API rate limits. Instead of searching backward from sinks like traditional SAST, agents trace data forward from attacker entry points. Every candidate finding then passes through a mandatory adversarial verification phase (Phase 2b) that explicitly tries to disprove it — the authors report ~50% of candidates are false positives. Candidates must clear five hard gates (designed behavior, reachability, attacker control, sanitization, new capability) backed by empirical source-code evidence before being written up.

The fix skill applies TDD-driven remediation: exploit demonstration → failing test → fix → regression check. It isolates each finding cluster in its own git worktree and enforces seven mechanical delivery gates before allowing a PR, including an anti-merge calculation (group_cost ≤ 0.6 × split_cost) to prevent unreviewably large PRs. The verify skill is a read-only independent check — four gates per fix, schema-validated output, fail-closed on drift. The Python runtime wraps these skills in a headless CLI using the Claude Agent SDK, with cold-start 429 backoff, stall detection, dual GitHub tokens for defense-in-depth, and JSONL audit streaming.

The entire system hard-gates on Opus 4.7+ — Sonnet and Haiku produce "noticeably worse results." Scans default to read-only; code execution requires explicit `--enable-bash` and `--no-read-only` flags, and Bash cannot be re-enabled via config file.
