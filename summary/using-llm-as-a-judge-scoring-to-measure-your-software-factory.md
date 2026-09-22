---
url: https://www.warp.dev/blog/using-llm-as-a-judge-scoring-to-measure-your-software-factory
title: "Using LLM-as-a-judge scoring to measure your software factory"
author: Warp (warp.dev blog)
date_fetched: 2026-09-22
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

Warp argues that organizations running coding agents should grade past agent sessions with LLM-as-a-judge scorers rather than rely on DORA-style outcome metrics alone. Agents leave a complete digital record of their work — prompts, tool calls, MCP results, PRs, specs — and that record can be scored after the fact to find where agents are deficient and drive systematic improvement. Scoring is presented as the foundation of "agentic self-improvement," where observer agents suggest changes to the factory based on how it has been scoring.

The piece lays out six primitives: (1) store full agent traces (inputs and outputs) in the cloud, API-accessible, so scorer agents can load them; (2) define scoring agents, each grading a single dimension — task compliance, efficiency, verbosity, code quality, or org-specific checks like internal MCP and Skill usage — as a judging prompt, classification rubric, and judge model; (3) pick a sampling strategy, since scoring itself costs money (Warp's internal factory spends about 3% of total token costs on scoring); (4) accumulate a corpus of scored runs to graph over time, catch regressions, and correlate changes in models, skills, and context with improvement; (5) automate the loop — self-improvement agents synthesize scorer output in batch and create updates to the factory definition; (6) use scorers as the grading instrument for benchmarking different model configurations against the factory.

The worked example is a redundant-test-creation scorer, built after Warp noticed that failure mode internally: a judging rubric, pass/fail output classifications, and a sampling rate (all shown in screenshots that did not survive the text fetch). Failures are investigated by reading both the coding agent's trace and the scorer's own trace — "since it's just another agent" — then adjusting the skills that drive the agents. The post is undated but postdates Warp's Sep 5, 2026 factory posts, and closes with a product pitch: early access to Warp Factories, with up to $10k in usage credits for qualified companies.
