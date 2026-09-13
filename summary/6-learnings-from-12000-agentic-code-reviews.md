---
url: https://blog.watson-labs.co.uk/6-learnings-from-12000-agentic-code-reviews/
title: "6 Learnings from 12,000 Agentic Code Reviews"
author: "Watson Labs (no byline)"
date_fetched: 2026-09-13
date_published: "n.d."
topics:
  - ai-code-review
  - guardrails-and-feedback-loops
---

# 6 Learnings from 12,000 Agentic Code Reviews

A metrics post from a Watson Labs-run software factory: 372 issues over 6 months at stealth Swamp customer Gymwasp, whose pipeline ("The Mandible") runs Plan → Build → Review → Ship with rework arcs. No human reads any code — the humans at the Plan and Review gates assert that agents understood the problem and make product calls. Claude Opus (4.6 → 5) was used throughout.

Review is seven parallel lanes — test coverage, clean code, frontend, DDD, security, accessibility, observability — each a separate agent process on its own model instance, so no lane can see what the others found. Each returns pass / warn / fail, the round's verdict is the worst of the seven, and every lane gets a narrow brief plus an explicit out-of-lane exclusion list ("without exclusion lists, you get the same finding seven times").

The six learnings: **65% of issues are merge-ready after a single review round**; the conditional pass rate halves after round 2 and flatlines near 87% cumulative, with **round 4 the elbow** where extra rounds introduce as much uncertainty as they fix; **29% of issues never clean-passed and shipped on warn** — a deliberate risk-tolerance dial; the **naive average understates cost by 44%** (3.31 rounds if you only count clean passes, 5.99 when warn-shipped issues are included — "the censored observations are data, not noise"); **per-issue review cost doesn't scale with volume** (median held at 4 rounds while shipping volume tripled, credited to the deterministic harness); and of **1,801 adversarial review rounds, 1,463 were on code vs 338 on plans** — agents write the code wrong far more often than the plan.

The second half is drift diagnostics: stagnating pass rates mean reviewers are oscillating, not converging (fix the briefs, not more rounds); one lane dominating fails means either genuine signal or an over-tuned brief; infrastructure flakes (3.7% of verdicts lost to agent crashes, 8.1% of rounds with an unassessed lane, 111 dead iterations) erode trust — "fix the brief, don't kill the lane." The closing frame: this is CI/CD optimisation pointed at a new target — instrument the pipeline, measure honestly with survival analysis rather than naive averages, find the elbow, retune, measure again.
