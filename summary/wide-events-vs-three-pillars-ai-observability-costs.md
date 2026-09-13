---
url: https://www.honeycomb.io/blog/wide-events-vs-three-pillars-ai-observability-costs
title: "Wide Events vs. Three Pillars: AI Observability Costs"
author: Nick Travaglini
site: Honeycomb
date_fetched: 2026-09-13
topics:
  - guardrails-and-feedback-loops
  - databases-and-data
---

Honeycomb's Nick Travaglini argues that agentic AI workflows make observability costs balloon — and that the culprit is the three-pillar model itself, not the volume of telemetry. Agents add new dimensions to track on top of the conventional ones (user ID, response codes, feature flags): which model ran, which skills were invoked, whether each tool call worked, what failovers the model "thought" to try, the initiating prompt. Under the three-pillar approach these facts get recorded three ways — a one-off prompt as its own time-series datapoint, the prompt text in a log, the trace initiation in a trace — each in a separate store with its own bill, plus a correlation burden the author says is unsolvable "because the distinction is required by hypothesis."

The cost analysis corrects a common instinct: retention is not where the money goes — ingest and query compute are. And because AI agents are non-deterministic, their outputs are statistical, so teams need *longer* retention to see behavioral patterns emerge at scale, meaning the standard "cut retention" lever backfires. The proposed alternative is the wide event model: one structured event (Honeycomb uses a flat, non-nested key:value schema) whose width is bounded only by practical ingest limits. Quoting Charity Majors' Observability 2.0 definition, metrics, logs, and traces become derived views over the same stored events — durations recorded directly, p95 computed at read time, traces composed via parent identifiers, and conversations of agents and sub-agents composed into an "Agent Timeline." Data is stored once and paid for once.

The argument's honest edge: the pillar model "can be cheaper in some cases due to preaggregation," but the uncertainty of whether your workload is one of those cases is itself the problem. Wide events require no trade between context and preaggregation, which is what makes spend predictable. The piece closes with a migration checklist — identify what must stay request-level versus what can be derived, model cardinality/retention/growth for forecasting, and connect token and model activity to telemetry volume — followed by the expected vendor pitch: Honeycomb was designed for wide events from the beginning.
