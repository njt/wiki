---
url: https://blog.shisa.ai/posts/melt/
title: "MELT: Testing Long-Lived Memory for Agentic AI"
author: Leonard Lin
date_fetched: 2026-07-18
date_published: 2026-07-13
---

# MELT Blog Post — Summary

Leonard Lin announces MELT (Memory Evaluation for Lifecycle Testing), an open benchmark runner for evaluating long-lived memory in AI agents, and makes the case that existing memory benchmarks test the wrong thing. Most benchmarks evaluate retrieval from frozen transcripts — essentially reading comprehension over a snapshot. Real memory, Lin argues, must handle correction (a user changes jobs), contradiction (conflicting claims that shouldn't be merged), temporal/as-of recall ("what was true then?"), consolidation (durable facts survive, ephemera fades), and abstention ("I don't know").

MELT separates two tracks: **scripted** (harness tells the system exactly what to write/supersede/decay — testing execution fidelity) and **agentic** (raw events, system decides what to remember — testing extraction policy). The native lifecycle suite has 13 behavioral axes and 217 cases in the full fixture. The same runner adapts LongMemEval, LoCoMo, and RHELM into a common report envelope.

The most important early finding is that **one "memory score" is not enough**. Preliminary runs show ShisaD dominating Memobase on lifecycle (95.4% vs 44.6%) but Memobase crushing ShisaD on LoCoMo QA (42.0% vs 6.2%). The ordering flips completely between benchmarks — and the per-axis breakdown reveals temporal/as-of recall as a shared weakness both aggregate scores hide. Lin emphasizes all results are preliminary single-run development data, not leaderboard claims.

Core design principles: zero PyPI dependencies (stdlib only), pluggable SUT adapter contract (any memory system can be evaluated), config-as-reproducibility-hash, preliminary-vs-final status gating, and per-case checkpointing with resume. Licensed Apache 2.0.
