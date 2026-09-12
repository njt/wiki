---
url: https://github.com/ayghri/i-have-adhd
title: "i-have-adhd"
author: ayghri (Ayoub G.)
date_fetched: 2026-08-14
date_published: unknown
license: MIT
topics:
  - agent-architecture
---

`i-have-adhd` is a coding-assistant skill that reshapes an AI agent's output for
a reader with ADHD. One `SKILL.md` file encodes ten output rules grounded in five
facts about ADHD cognition: small working memory, the knowing-doing gap,
cold-start paralysis, time blindness, and dopamine scarcity. The rules: lead
with the next action, number multi-step work, end with one concrete next step,
suppress tangents, restate state every turn, give specific time estimates, make
wins visible, report errors matter-of-factly, cap lists at five, and forbid all
preamble, recap, and closers. The rules persist for the whole session until the
reader says "stop adhd mode".

The repo is more than a prompt. The same ruleset is packaged for ten-plus
harnesses — Claude Code, Codex, Pi, Gemini CLI, Kimi, Qwen, Cursor, GitHub
Copilot, Zed, Hermes — via small per-harness manifests, with three always-on
routes layered on top. The deepest integration is a Pi extension
(`extensions/i-have-adhd.ts`) that injects the ruleset once per session as a
typed message, re-injects it after compaction, and restores on/off state from
saved session state. A Claude Code `SessionStart` hook makes the mode always-on
behind an opt-in flag file (`~/.claude/.i-have-adhd-always`).

The project also ships a paired eval harness (`scripts/run_evals.py`, `evals/`)
that treats output shaping as a measurable quality regression: baseline vs.
candidate responses judged blind on a weighted rubric (correctness 35%, autonomy
25%, actionability 20%, safety 10%, concision 10%), gated behind a release check
that the candidate must beat baseline on.
