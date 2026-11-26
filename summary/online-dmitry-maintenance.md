---
url: https://ai.engineer/code/2026/talks/online-dmitry-maintenance
title: "Online Talk: Maintaining Large AI Coding Projects (Dmitry)"
author: Dmitry (speaker; surname not stated in transcript)
date_fetched: 2026-11-26
date_published: 2026 (ai.engineer talk)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A machine-generated transcript of an ai.engineer talk on maintaining a large, mostly agent-written production system — an order-processing "pros" (prompt/instruction) engine running in 110 countries with ~10,000 scenario instructions. The speaker opens with the Thanos frame: coding agents made code cheap, and entropy even cheaper; his first response was a full rewrite ("the snap"), which failed because the agents kept writing and the mess grew back. The talk's thesis is the third option: stay under the line of unreviewable change with small weekly sweeps and feedback loops built into the system, structured as six "stones" (questions): time, space, reality, mind, power, soul.

Key content: "right" is decided by people, not tests — a mutation-testing experiment on the pros instructions turned almost nothing red, so the tests were not testing the thing that mattered; human judgment is spent deliberately, with developers tracing whether a "check with the team" step is truly a human decision. Instruction and code drift is fought with a hand-built "compiler" that lints the pros, and with "walks": repeated agent paths through the instructions get promoted into workflow tools and the instructions that walked them are deleted — but only when a walk repeats and agrees across countries. Control is reframed as context discipline (the runtime, not the model, decides what the model sees; a model with prices visible invented a pricing check) plus write-time gating: five walls in front of every write, a sixth checking it landed, journal-based idempotency.

The most personal chapter is on the bottleneck: the speaker became "the one person every change waits on" and burned out; the first gauge is himself — when he's tired and can't explain what changed, the process is broken even if the dashboard is green. Instruments (blast-radius replay of new checks over thousands of past sessions, ratchets that only go down, agent-vs-agent verification with a checklist and no access to the author's reasoning) replace reading every diff. He also flags that agents quietly skip work when context fills: a backlog that looked ~100% covered was really only ~25% closed, so only countable evidence changes his mind. The talk closes with the Conway corollary: "the system copies whoever corrects it," and an admission that his own biases are baked into the system's skills and memory. Ends with 18 questions (three per stone) as a take-home diagnostic.
