---
url: https://www.cockroachlabs.com/blog/experiment-running-hospital-code/
title: "Experiment: Running Hospital Code"
author: Cockroach Labs
date_fetched: 2026-10-24
date_published: 2026
topics:
  - agent-orchestration
  - ai-code-review
---

Cockroach Labs describes "MOLT Sinai," a multi-agent software factory modeled on a teaching hospital, which produced over a million lines of code in five months across four repos. The trigger case: IBM Db2 support for their MOLT migration tool landed in two days for $4,172 in tokens, versus 9 months and ~$160K for the equivalent Oracle work in 2024 — 164x faster, 38x cheaper — with no human writing any code.

The hospital metaphor structures the whole pipeline: issues are patients, merging is discharge, humans are Chiefs of Medicine. It runs on GitHub Actions with labels as the state machine — each stage is a workflow that fires on a "label in," and the finishing agent applies the "label out" that triggers the next stage. Roles include Fellows (diagnose and treat), Review Attendings (find fault, not fix), a Discharge Nurse who audits whether the review actually happened, plus a Charge Nurse, Infection Control, Safety and Research departments. The core rules are strikingly conservative: no code before a reviewed plan, no weakening tests (test changes must be separate reviewed commits, and reviews start by disabling the change to confirm new tests fail), I-PASS structured handoffs on escalation instead of retrying, and a human-only precedent log agents must consult before escalating.

Key numbers: 1,238 PRs in under five months, 1,299 issues of which nearly half were filed by agents for agents; $135K in tokens (~$84 per issue); plans rejected under 10% of the time but catching high-value bugs. The costs of the model are also documented honestly: bureaucratic loops (a one-line UI copy edit took five rework rounds), non-converging review loops that required a circuit-breaker (hand off to an Attending after four consecutive rework rounds), and 23% of a 100,000-word skill corpus identified as redundant — the prompts rotted like code. Notably, the Db2 code was never human-reviewed, and the team's stated next step is verifying it is correct. The hospital also built itself: 85% of the Sinai framework repo's 1,570 commits were authored by the pipeline running on its own bugs.
