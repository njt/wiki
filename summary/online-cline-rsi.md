---
url: https://ai.engineer/code/2026/talks/online-cline-rsi
title: "Recursively Self-Improving Coding Agents (Era, Cline)"
author: Era (founding engineer at Cline)
date_fetched: 2026-11-26
date_published: 2026 (talk, exact date not stated)
topics:
  - agent-architecture
  - coding-agents-and-frameworks
---

A talk transcript by "Era," a founding engineer at Cline, on recursively self-improving (RSI) coding agents. His central worked example: Cline automated the manual "hill climbing" they used to do on model releases. When Kimi K3 scored 77.5% on Terminal Bench 2.1 against a claimed 85%, they ran a long-running loop — GPT-5.6 (transcript garbles it as "GPD 5.6") as the improving meta-agent, K3 running the evals — for 17 hours. The meta-agent bucketed failures (loop detection, background-process grinding, file-index issues), edited the harness, re-ran the 89-task eval, and raised the baseline 27% score substantially without human intervention.

He frames this as one node in a stack-wide RSI loop: pre-training data curation, post-training, eval generation, inference (citing OpenAI's claim that GPT-5.6 rewrote its own production kernels, cutting serving cost ~20%), and even hardware — he describes letting coding agents optimize the firmware and models of a small consumer AI gadget overnight. His rule: anything with a verifiable loop can be optimized.

The talk ends with career advice aimed at engineers feeling the ground shift: "move closer to the RSI loop." Most niche agent products are "one model release away from being wiped out"; better to work on the improvement pipeline itself — evals, data, inference, hardware — and avoid problems that scale alone will solve.
