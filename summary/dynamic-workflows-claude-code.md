---
url: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
title: "A Harness for Every Task: Dynamic Workflows in Claude Code"
author: Thariq Shihipar and Sid Bidasaria
date_fetched: 2026-07-29
date_published: 2026-06-02
---

Anthropic blog post announcing dynamic workflows in Claude Code — a feature
that lets Claude author custom orchestration harnesses on the fly, written as
JavaScript files with special primitives for spawning and coordinating
subagents.

The default Claude Code harness runs planning and execution in one context
window, which works for coding but breaks down over long-running, parallel, or
adversarial tasks. Three failure modes emerge: agentic laziness (declaring
partial progress complete), self-preferential bias (favoring its own outputs
when verifying), and goal drift (losing fidelity across turns, especially after
compaction). Workflows combat these by isolating subagents in their own context
windows with focused goals.

The article describes six composable patterns: classify-and-act,
fan-out-and-synthesize, adversarial verification, generate-and-filter,
tournament, and loop-until-done. It covers use cases from migrations and deep
research to triage at scale, evals, and model routing. It also notes when *not*
to use workflows — they burn more tokens, and parallelism must earn its
coordination cost. Tips include pairing workflows with `/goal` and `/loop` for
repeatable runs, setting token budgets, and saving workflows to
`~/.claude/workflows` or distributing them via skills.

*Sources: [[raw/dynamic-workflows-claude-code]]*
*Last updated: 2026-08-01*
