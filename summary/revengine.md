---
url: https://www.laude.org/updates/lab-notes/revengine
title: "RevEngine: Scaling Code Review at Laude"
author: Laude
date_fetched: 2026-10-21
date_published: 2026-10-20
topics:
  - ai-code-review
  - coding-agents-and-frameworks
---

Laude built RevEngine, an open-source Apache-licensed GitHub Action for AI code review, born from the flood of outside pull requests that followed the release of their agent harness Headlong. A minimal harness plus a strong model — their recurring thesis — beat off-the-shelf review tools for their needs.

The design is simple: give the model read-only repo access to review a PR against the project's rules and design docs, then run a second pass that rechecks each finding against code context and drops the unsupported ones. Deduplicate, rank by severity and confidence. With Opus 5.5, a typical review takes ~3 minutes and ~$1 in tokens.

On GitHub's ReviewBench (~2,600 human-validated issues across 219 PRs in 187 repos), RevEngine topped the leaderboard when results went public on 2026-10-16, raising the top F1 from 48% to 59.6%, and it was the fastest agent on the board — the only point on the F1-vs-time Pareto frontier.

Their ablations are the interesting part: a strong model sets the baseline, but the harness makes reviews thorough. Adding the harness raised F1 for every model tried (up to +14 for Sonnet), let Opus catch ~a third more real issues, and lifted high-severity recall from 32% to 55%. Harness gains were larger for stronger models. Surprisingly, Sonnet nearly matched Opus under the harness at half the cost — Opus kept only a slight lead on high-severity findings.

Lessons: write the project's rules down so an automated reviewer can judge whether a change both works *and belongs*; match reviewer strictness to context (agents don't tire of fixing ten nits, humans do); review is cheap compared with human attention; and keep a person on the riskiest parts of the codebase.
