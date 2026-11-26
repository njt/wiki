---
url: https://ai.engineer/code/2026/talks/online-harvey-agent-platform
title: "Online Harvey Agent Platform (Joey Wang)"
author: Joey Wang (reading edition, ai.engineer)
date_fetched: 2026-11-26
date_published: 2026 (exact date not verified)
topics:
  - agent-orchestration
  - agent-architecture
---

Joey Wang's talk on **Spectre**, Harvey's internal agent platform, is a field report on running coding agents as shared infrastructure rather than as a per-engineer toy. The scale: more than one agent started every minute around the clock, thousands of agent-written pull requests, and usage well beyond coding — Slack data questions, a marketing colleague prepping a product video.

Wang's first argument: Harvey runs engineers' favourite agents rather than replacing them. Spectre supplies company secrets, a real test stack, alerts, tickets, shared policies, and capacity. The valuable piece is the **follow-up loop** after the first patch — overnight alert to proposed fix, real stack validation, proof attached to the PR, CI-failure retry — with human approval kept distinct from patch generation and merging. If you can build only one thing, build that loop.

Second: make every product action programmatically callable. A CLI exposed Spectre's operations so laptop agents, sandboxed agents, and CI could all drive the same execution layer — which also means a coordinator agent can dispatch hundreds of milestone tasks, so interface design silently sets your worst-case load.

Third, the failures: an alert storm spawned ~900 duplicate agents on one bug (fixed by deduplicating alerts against in-progress investigations), then a legitimate engineer's CLI burst created honest overload requiring **admission control** — a queue that separates getting a place from getting a worker, reserves room for interactive work, limits concurrency per person, and allows borrowing. Wang is candid that shipped-together changes can't be individually credited. The talk ends unresolved: after execution speeds up, the bottleneck moved to merge waits on a moving main branch, and unattended automations need budgets, expiry, and failure policies. Spectre's value is owning everything between "agent wrote a patch" and "change is reviewed and merged."
