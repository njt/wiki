---
url: https://claude.com/blog/ai-code-migration
title: "How Anthropic runs large-scale code migrations with Claude Code"
author: Michael Segner (based on migrations run by Jarred Sumner and engineering teams across Anthropic)
date_fetched: 2026-07-25
date_published: 2026-07-16
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Anthropic's playbook for using Claude Code to run large-scale code migrations,
built from two real-world examples: migrating Bun from Zig to Rust (~1M lines in
under two weeks, $165K in API cost) and migrating a Python codebase to TypeScript
(165K lines over a weekend).

The central philosophy is shifting effort from fixing individual bugs to fixing
the process that produces the code. A six-step pipeline runs from building a
"judge" (portable test harness) through rulebook creation, stress-testing those
rules with mini-migrations, fan-out translation by many agents, then
compile/run/behavior-match loops with progressively less human judgment.

Key practices: adversarial review paired with mechanical verification, smaller
models for high-volume implementation and largest models reserved for reviewers
and rule-writers, and a mechanical work queue where "done" means the output file
exists on disk. Human attention stays on patterns, not individual failures.

The Bun results in production: memory dropped from 6,745 MB to 609 MB in one
benchmark, binary 19% smaller, and 2–5% faster across real-world workloads.

---
*Sources: [[raw/ai-code-migration]]*
*Last updated: 2026-08-01*
