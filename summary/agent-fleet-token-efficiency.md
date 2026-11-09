---
url: https://alexmarquardt.com/ai-tools/agent-fleet-token-efficiency/
title: "Agent Fleet Token Efficiency"
author: Alex Marquardt
date_fetched: 2026-11-09
date_published: ~2026-11 (data through W45, 7 November 2026)
topics:
  - claude-code
  - agent-orchestration
---

Alex Marquardt runs a fleet of up to a dozen background Claude Code agents on a multi-service search codebase (~1,400 files, 136 CI jobs, 590–870 merged PRs/week across four repos) and reports ten weeks of token accounting measured from the transcripts Claude Code keeps on disk. The headline: tokens per changed line fell 3.3x (2.5x price-weighted), tokens per merged PR 5.5x, and merged PRs per week rose 4.4x on roughly the same spend.

The framing that carries the article: agents are charged for context re-reading, not for writing. Total tokens ≈ number of calls × average context per call, and cache reads are 94–98% of all tokens. Six changes each attack one side of that product: measure per merged PR from `~/.claude/projects/` transcripts (counting each request once); end each agent after one task with a written handoff and a `PreToolUse` hook that enforces a context ceiling (median context per request fell 218k → 101k on day one); keep the prompt cache warm — idle cache expiry was 31% of agent usage and fell to 0.7% by returning before the 270-second sub-agent cache lifetime; turn rules into code; replace the model-driven coordinator with a small program (coordinator tokens per merged PR down to 7% of baseline); and make the codebase navigable for an agent that is "a new developer every time it starts."

The honest sections are as valuable as the wins: output tokens per PR barely fell (only ~10% of model output is file edits; half is reasoning), rework per PR rose slightly (0.8 → ~1 fix round), the fixed first-turn context actually grew, and a table of what did not pay includes shorter briefs (0.6% of use — never mattered) and blind tool-output caps. The recurring principle: "don't make the model remember; make the system remember."
