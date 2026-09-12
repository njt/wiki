---
title: "Trycycle"
url: https://github.com/danshapiro/trycycle
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

A skill for Claude Code, Codex CLI, Kimi CLI, and OpenCode that plans, strengthens, and reviews code automatically. By Dan Shapiro.

Philosophy: take any request of any size or complexity, avoid asking the user questions, prioritize zero bugs even if it takes a lot of time and tokens.

How it works: Trycycle is a hill climber. Writes a plan, sends to a fresh planning issue finder with same task input and repo context. If reviewer finds plan-breaking issues, Trycycle deepens on the same reviewer, then hands findings memo to a fresh planning synthesizer that rewrites the plan holistically. Fresh reviewer checks, repeating up to five review/synthesis rounds. Once plan is locked, builds test plan, builds code, sends to fresh reviewer, turns review into structured observation packet, fixes what the packet shows, repeats up to eight rounds.

Key insight: each review uses a fresh reviewer with no memory of previous rounds, and each planning round spawns a fresh agent, so stale context never accumulates. Plan reconsideration runs after 4th review round and every 2 rounds after that.

Adapted from superpowers by Jesse Vincent. Dark factory approach inspired by Justin McCarthy, Jay Taylor, and Navan Chauhan at StrongDM.

Works with Deepseek v4 via OpenCode for cost-effective operation.