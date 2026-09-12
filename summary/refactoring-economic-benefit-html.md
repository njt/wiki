---
url: https://martinfowler.com/articles/exploring-gen-ai/refactoring-economic-benefit.html
title: The Economic Benefit of Refactoring
author: Martin Fowler (Thoughtworks technologists series)
date_published: 2026-07-30
date_fetched: 2026-08-07
series: Exploring Gen AI
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

# The Economic Benefit of Refactoring — Summary

Martin Fowler ran a controlled experiment to measure whether refactoring agent-generated code reduces the token cost of future changes. The subject was a 17,155-line Rust data access layer, entirely written by Claude Code without human review.

The experiment design exploited a unique property of AI agents: unlike humans, they never learn. Fowler could prompt a fresh sub-agent to make the same representative change after each refactoring step without contamination from prior experience. He measured input tokens, output tokens, and time for each attempt across 15 increasingly aggressive refactorings.

The result: input tokens dropped from 159,564 to 27,360 — an **83% reduction** — not from deleting code (total lines stayed roughly constant) but from decomposition that let the agent read only the files it needed. The saving came from the agent successfully identifying smaller and smaller relevant subsets of code. Output tokens were largely unaffected, suggesting refactoring helps reading far more than writing.

Claude was notably poor at autonomous refactoring — it needed explicit human guidance at every step, frequently botched mechanical edits with grep/sed scripts, and missed the most valuable single refactoring on the first pass. The experiment took ~8 hours mostly unattended. The upper bound for tokens consumed *by* the refactoring itself was 5 million, though Fowler couldn't separate experiment-design cost from refactoring-execution cost.

The finding is that refactoring for agents has a measurable, repeatable economic return: spend tokens now on structure to save tokens on every future change. At $3/MTok (Sonnet 5), the per-change savings are modest (~$0.40), but they compound across every subsequent change touching that module.
