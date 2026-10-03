---
url: https://www.mitchelsellers.com/blog/article/your-coding-agent-needs-observability-too
title: "Your Coding Agent Needs Observability Too"
author: Mitchel Sellers
date_fetched: 2026-10-03
date_published: 2026-09-28
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

Mitchel Sellers argues that teams adopting AI coding agents are running a new production workflow blind — they see sessions start and finish, and usage numbers, but cannot explain why a session went sideways, burned tokens, or produced a bad result. The trigger for the piece is GitHub's September 22, 2026 announcement of OpenTelemetry support for the Copilot app, exportable via enterprise managed settings to any OTLP endpoint.

His core distinction is adoption metrics versus execution telemetry: dashboards counting active developers and suggestion-acceptance rates answer management questions, not troubleshooting questions. Traces, metrics, and events exported from agent sessions — model calls, tool usage, accept/reject events, token burn — let a team ask whether a session spent its time waiting on the model or thrashing on tools, and whether a config change improved reliability or just moved the problem.

The operational advice is deliberately conservative: keep `captureContent` (prompts, responses, tool arguments) disabled and locked by default because it can carry source code and secrets; treat metadata as sensitive too, since it still reveals usernames, timing, and working patterns; decide on credential delivery, retention, and query access before enabling export; and define a handful of operational questions before building dashboards — explicitly warning against turning developer activity into a leaderboard. He closes by stressing that telemetry complements rather than replaces guardrails (permissions, branch protection, sandboxing), and recommends a small pilot rollout framed as an engineering experiment.
