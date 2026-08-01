---
url: https://cursor.com/blog/cloud-agent-lessons
title: "What We've Learned Building Cloud Agents"
author: Josh Ma
date_fetched: 2026-07-11
date_published: 2026-06-02
---

Cursor's engineering team reflects on the challenges of building coding agents that run
on remote infrastructure rather than a user's laptop. The post is unusually concrete:
real reliability numbers, named infrastructure choices, and honest admissions about
early failures.

The central claim is that **the development environment IS the product**. On a laptop,
the agent inherits the user's shell, tools, keys, and repo state. In the cloud, all of
that must be reconstructed — and incomplete environments produce subtle degradation in
output quality, not explicit errors. Cursor built what amounts to enterprise IT for
agents: VM hibernation, checkpoint/restore, secret redaction, and credential management.

**Durable execution** was the second hard lesson. Cloud agents face inference provider
outages, pod replacements, and node failures that local agents don't. Cursor's initial
work-stealing architecture operated at roughly one nine of reliability. Migrating to
Temporal for durable execution primitives (retry, scheduling, node-failure survival)
took them past two nines. They now process over 50 million actions per day across 7
million unique workflows, and over 40% of Cursor's PRs come from cloud agents.

The third insight is **decoupling agents, machines, and conversation state** into three
independent components. The agent loop runs in Temporal, machine state can migrate
across infrastructure, and conversation state streams to clients via an append-only
storage layer that handles retries gracefully.

The post also describes a counterintuitive evolution: as models improve, harness code
should **get out of the way** rather than double-check every action. Cloud agents get
prompts encouraging more autonomy because blocking on human input is far more expensive
than for local agents. The forward-looking goal is self-healing environments — agents
that can diagnose and fix their own missing secrets, blocked network access, and broken
toolchains.
