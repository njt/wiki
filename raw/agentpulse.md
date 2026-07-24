---
url: https://prove-ai.github.io/agentpulse/
title: AgentPulse
author: Prove AI
date_fetched: 2026-07-25
date_published: 2026
---

# AgentPulse

AgentPulse is an open-source drift investigation tool designed for multi-agent systems. It helps identify which agent drifted, why, and what to check next.

**Tagline:** "Traces tell you what happened. AgentPulse shows you where to investigate."

**License:** MIT License, © 2026 Prove AI

## Supported Integrations

OpenAI SDK, Anthropic SDK, AutoGen, LangChain, LangGraph, and Claude Code (via MCP).

## How It Works

1. **Instrument** — Users add two lines at startup. It auto-instruments OpenAI, Anthropic, LangChain, and AutoGen.
2. **Capture** — LLM calls, agent turns, tool calls, and handoffs are stored locally in one SQLite file per project. Core tracing works without a cloud account or hosted backend.
3. **Detect drift** — The system compares behavior across runs and versions, flagging agent, handoff, and route drift when signals leave expected ranges.
4. **Investigate** — Users follow drift upstream to its likely source, with supporting signals, related changes, confidence, and suggested next checks.

## Investigation Methodology

When outcomes degrade, AgentPulse compares every component against a baseline and traces the agent graph to find drift origin.

Example investigation path:
- Symptom: critic → "success −70 pp vs baseline"
- Root cause: writer → "output drifted · input stable"
- Change log: "writer prompt + model changed at run 18 · flagged as the likely cause"
- Stable components: researcher input (stable, cleared), analyst input (stable, cleared)

## Four Rules for Root-Cause Findings

1. Baseline every metric, per agent, handoff, and route
2. Only open an investigation on a sustained outcome breach (not a single noisy spike)
3. Walk the graph upstream and stop where inputs are stable but output drifted
4. Correlate prompt, model, and tool changes that could have caused it

## Features

- Overview dashboard showing runs, success rate, active drift signals, top signals, version snapshots with per-version success and cost
- Drift-focus view with "Likely cause identified" and "Related change detected" badges
- Run detail screen showing parallel workflow with execution timeline, overlapping agent turns, fan-out agent graph
- Execution timeline and interactive agent graph (not raw logs): tokens, latency, cost per turn; parallel branches, bottlenecks, join times; handoff payloads and downstream outcomes on hover

## Claude/MCP Integration

AgentPulse ships an MCP server, enabling Claude Code and Claude Desktop to triage drift, compare releases, and propose next checks conversationally. Three bundled skills:
- `get_todays_finding` — Active drift findings as root-cause-led investigation cards
- `get_version_comparison` — Identifies which release introduced a change that broke an outcome
- `get_next_check_steps` — Recommended next investigation steps per finding

## Links

- GitHub: github.com/prove-ai/agentpulse
- Contact: leyla@proveai.com
