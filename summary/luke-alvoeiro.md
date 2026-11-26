---
url: https://ai.engineer/code/2026/talks/luke-alvoeiro
title: "Autonomy in Checkpoints: Luke Alvoeiro on Software Factories and Workstreams"
author: Luke Alvoeiro
date_fetched: 2026-11-26
date_published: 2026
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

Luke Alvoeiro, who leads Factory's core agent team, argues that teams getting the most from coding agents are not those handing over the most work on day one, but those ramping autonomy gradually, "one checkpoint at a time," until the system resembles a software factory: a 24/7 loop that takes in signals (Sentry alerts, Slack complaints), turns them into changes, validates them, and learns from production.

Factory's first big multi-agent product, Missions, worked only for clear, verifiable goals and failed three ways: it assumed full up-front planning with no interactive surface; adversarial review at every milestone ballooned scope (60 features to 200+) and cost; and context stayed trapped with one person. Their fix borrows from operating systems — scheduling, safe concurrency, shared state.

The rebuilt stack: a tuned harness with programmatic tool calling (150 Linear ticket moves go from ~30 inference requests / 1M tokens to one program / ~60k tokens); reusable sandbox "environments" with a single control plane running in the customer's VPC; and "workstreams" — standing goals with an identity, environment, and memory, orchestrated by a scheduler-like agent that delegates to owned areas rather than a rigid DAG (which they tried first and abandoned).

Autonomy is levelled L1 (human review at spec, PR, and merge) up to L4 (automerge), with the owner raising levels only as observed quality earns it. The gate metric is the "regret-free merge rate" — merged changes never needing revert or follow-up fix — measured for droid-written and human-written PRs side by side, with self-driving cars as the analogy. Validation is platform-level: code review, security review, QA against live previews, and a risk judge whose policy comes from the mainline branch so the change under review cannot rewrite its own rules. Shared filesystem memory means one agent's learning is reusable and "nothing starts cold."

In production Factory shipped ~800 PRs in one weekend, 99% droid-written and all droid-reviewed; 52% of PRs now come from workstreams, some fully autonomous (a Slack bug-fix workstream merges 53 PRs/week, flaky-test one ~40). His test for whether you have a factory or just faster engineers: can you name and track your signals?
