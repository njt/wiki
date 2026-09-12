---
url: https://www.adamhjk.com/blog/a-practical-guide-to-reducing-token-spend
title: "A Practical Guide to Reducing Token Spend"
author: Adam Jacob
date_fetched: 2026-07-25
date_published: 2026-07-16
topics:
  - agent-orchestration
  - ai-code-review
---

Adam Jacob presents a case study and practical guide for dramatically reducing LLM token consumption in AI-assisted code review by replacing coordinator-agent architectures with deterministic workflow code.

The trigger is David Cramer's report of spending over $10,000/week on tokens at Sentry, driven by an AI code-review skill (Garfield) that used a coordinator agent dispatching 23 sub-agents — consuming ~4.5M tokens and ~12 minutes per run. Jacob identifies the root problem as putting "the LLM in the hot path" for work that doesn't need AI intelligence.

His solution, a "swamp workflow," replaces the non-deterministic coordinator loop with deterministic code that calls agents only where intelligence is required. Sub-agent results are stored as versioned, typed data for visibility and to avoid costly re-work. The rebuilt skill uses ~500K tokens (8× reduction), ~6.5 minutes (2× faster), and 3 agents instead of 23.

Jacob provides a four-step guide: (1) understand the existing skill by having an LLM summarize it, (2) translate to a swamp extension — plan first, then implement, checking that the plan captures intended outcomes not implementation details, (3) black-box test both versions side by side with AI-built test suites, and (4) refactor. He notes the swamp version notably "fails closed" (reports unresolved findings) where the original "failed open" (declared success despite defects) — a desirable fail-secure pattern.

The core principle: "You use the Agent to build the program that minimizes the need for the Agent itself."
