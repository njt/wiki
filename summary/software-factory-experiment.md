---
url: https://lethain.com/software-factory-experiment/
title: "Trying the Software factory pattern"
author: Will Larson (lethain)
date_fetched: 2026-09-25
date_published: 2026-09-25
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Will Larson (CTO at Imprint) describes adopting the "software factory" pattern — looping on a broad goal and letting the harness drive progress — as the latest step in Imprint's 2026 adoption arc. That arc is itself the most valuable part of the piece: Claude Code daily for engineers in January, for everyone by March, workspace-level (not repo-level) local development with ~10 independent checkouts enabling cross-repo PRs by April, a company-wide migration from Jira to Linear in June to give agents a legible task system, then the "Agent Fleet" orchestrated harness (in the vein of Stripe's Minions) in July.

The first factory implementation is deliberately modest: a `/linear-project-loop` agent skill that audits a Linear project's scaffolding (an RFC in Notion, a Datadog dashboard or Snowflake queries measuring progress), iterates with the user to create what's missing, reviews metrics and issues, adds new issues, works non-blocked tasks (PRs, review pings, clarifying questions), and reruns the goal audit when the project description goes stale. Larson runs it locally for now but expects to move it to the same orchestrated harness used for one-off tasks.

Two payoffs stand out. First, the factory pattern forces the human to stop hoarding project state: agents previously couldn't judge whether work was going in the right direction or whether necessary tasks were missing, because the goal context lived in Larson's head. Second, it enables cheap post-release vigilance — his passkeys project had gone unmonitored for months, and a low-frequency "factory mode" would catch adoption spikes or error-rate turns immediately. His closing observation: all the pieces compound only in the presence of the others — Datadog/Snowflake MCP for goal-tracking, Linear as the single source of work state, and an orchestrated harness that works independently of his laptop.
