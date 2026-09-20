---
url: https://apievangelist.com/2026/08/27/your-agent-should-run-its-own-observability-stack/
title: "Your Agent Should Run Its Own Observability Stack"
author: Kin Lane
date_fetched: 2026-09-20
date_published: 2026-08-27
topics:
  - guardrails-and-feedback-loops
  - mcp-and-tool-protocols
---

Kin Lane argues that an agent's telemetry — tool calls, retries, reasoning-loop costs — should stay inside the boundary the agent runs in, shipped to self-hosted Prometheus, OpenTelemetry, and Tempo rather than a commercial observability vendor. His argument is deliberately not the usual cost one: agent telemetry is a "transcript" of what the model considered and rejected, so egressing it is a governance decision most teams make by default because monitoring is something you buy.

The surprising empirical core: when he actually called the APIs, Prometheus's single query endpoint plus PromQL turned out to be a *better* agent affordance than Datadog's 290 REST endpoints, despite scoring lower on his catalog's agent-readiness numbers. Prometheus also lets an agent discover its entire schema (labels, metric names, types, help text, cardinality) before writing a query — legible because it was designed for operators who introspect, not for a UI.

He then audits the self-hostable stack an agent would need (observability, memory, sandboxing, serving, orchestration, identity, storage) and finds most of it already catalogued but poorly scored by rubrics built for hosted SaaS. The gap he commits APIs.io to closing: nobody publishes a license-aware, deployment-aware index telling an agent what container to pull, what port to poll, and which license governs it — the difference between OpenBao and Vault matters when an agent is told to "install an open-source secrets manager."
