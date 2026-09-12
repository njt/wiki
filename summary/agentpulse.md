---
url: https://prove-ai.github.io/agentpulse/
title: "AgentPulse"
author: Prove AI
date_fetched: 2026-07-25
date_published: 2026
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

AgentPulse is an open-source (MIT) drift investigation tool for multi-agent
systems. Its tagline: "Traces tell you what happened. AgentPulse shows you where
to investigate."

It instruments LLM calls, agent turns, tool calls, and handoffs with two lines
of setup, storing everything locally in a per-project SQLite file — no cloud
account required. Supported SDKs include OpenAI, Anthropic, AutoGen, LangChain,
and LangGraph, plus Claude Code via an MCP server.

The core workflow is four-stage: instrument, capture, detect drift, then
investigate. Drift detection compares behavior across runs and versions,
flagging agent, handoff, and route drift when signals leave expected ranges.
Investigation follows a set of four rules — baseline everything, only open on
sustained breaches, walk the graph upstream to where inputs are stable but
output drifted, and correlate prompt/model/tool changes.

The Claude/MCP integration bundles three skills: `get_todays_finding` (active
drift findings as investigation cards), `get_version_comparison` (identify
which release broke an outcome), and `get_next_check_steps` (recommended next
checks per finding).

The interface includes an overview dashboard, a drift-focus view with
cause/change badges, and a run detail screen with execution timeline and
interactive agent graph showing tokens, latency, cost, parallel branches,
bottlenecks, and handoff payloads.
