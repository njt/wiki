---
url: https://ai.engineer/code/2026/talks/zach-lloyd
title: "Factory Engineering (Zach Lloyd, ai.engineer 2026)"
author: Zach Lloyd
date_fetched: 2026-11-26
date_published: 2026
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Zach Lloyd (Warp CEO, ex-Google Docs) argues interactive local agent prompting is a transitional stage capped by human throughput and plagued by cost, security, and governance problems at team scale. The destination is the "software factory": a **code-defined, closed-loop system for continuously measuring and improving cloud coding agents**.

His definition has three parts: (1) encode the whole factory in a declarative, Terraform-style language — agents, triggers, workflows, MCPs, where they run; (2) run a continuous closed loop from idea intake through triage, implementation, verification, and monitoring; (3) make everything measurable — no "vibes-based" deployment of agents.

He lays out an infrastructure stack: factories-as-code at the bottom (enabling branching, rollback, and agent-suggested factory changes), a data/context layer recording every agent conversation and artifact, a multi-platform sandboxing layer, a pluggable inference layer (multiple models and harnesses as "a degree of freedom to test"), orchestration, and an API/MCP-first surface integrating Slack, Linear, Jira, and GitHub. The measurement tooling includes online evals scored by a second agent (e.g. a redundant-tests detector) and benchmarks replayed on your own real data across model, harness, and configuration variants. The endpoint is **self-improvement**: agents diagnose failure patterns in their own runs and submit PRs against the factory definition itself.

The closing reframe: agents are a new engineering primitive, so the new skill set is "factory engineering" — evals, benchmarks, A/B testing the factory — alongside traditional product engineering.
