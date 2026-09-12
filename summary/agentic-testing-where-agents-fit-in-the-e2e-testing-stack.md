---
url: https://slack.engineering/agentic-testing-where-agents-fit-in-the-e2e-testing-stack/
title: "Agentic Testing: Where Agents Fit in the E2E Testing Stack"
author: Sergii Gorbachov (Staff Software Engineer at Slack)
date_fetched: 2026-06-21
date_published: 2026-06-11
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

# Agentic Testing: Where Agents Fit in the E2E Testing Stack

Sergii Gorbachov, Staff Software Engineer at Slack, reports on 200+ agentic E2E workflow runs using Playwright MCP, Playwright CLI, and agent-generated Playwright tests in test workspaces with non-production data.

## Core Thesis

"Tests enforce journeys. Agents verify goals." Traditional E2E tests validate a specific UI journey (click → click → type → assert), while agent-driven tests validate whether a goal can be achieved from a natural language instruction (goal → agent adapts → verify result). Agent-driven E2E tests add a new exploratory layer on top of traditional testing rather than replacing deterministic tests entirely.

## Experiment Design

Three execution models tested across two test flows (Thread Reply: ~15-20 steps; Search Discovery: ~25-30 steps), 20 runs per configuration:

| Model | Description |
|-------|-------------|
| Agent + Playwright MCP | Agent interacts with browser via Model Context Protocol, persistent DOM context |
| Agent + Playwright CLI | Agent runs Playwright CLI commands via shell, snapshot-per-step |
| Generated Playwright Tests | AI generates deterministic Playwright code, iteratively refines until passing |

Models used: Claude Sonnet 4.5 (MCP/CLI agents), Claude Opus 4.6 (generated tests and cost analysis), Claude Haiku 4.5 (cost comparison). Execution via Claude Code (`claude -p`).

### Results Summary

| Approach | Thread Reply Failure | Search Discovery Failure | Avg Runtime |
|----------|---------------------|-------------------------|-------------|
| Agent (Playwright MCP) | ~0% | ~12% | ~5–8 min |
| Agent (Playwright CLI) | ~12% | ~20% | ~9–11 min |
| Generated Playwright Tests | ~8% | ~48% | ~3 min |

### Key Findings

1. **MCP dramatically outperforms CLI**: Near-zero failure on simple scenarios vs 12-20% for CLI. MCP's persistent live DOM view outperformed CLI's snapshot-per-step approach. CLI failures were predominantly authentication and navigation issues — "failures were caused by the execution layer rather than the agent's reasoning."

2. **Generated tests degrade on complexity**: ~48% failure on complex workflows. Tests typically progressed through 70-80% of the flow before breaking on a final interaction or assertion. Failures driven by "variability in UI state and abstraction mismatches."

3. **Cost is dominated by context accumulation**: $15-30 per execution. Token usage: MCP (Sonnet 4.5) ~3.5M tokens/40 turns, CLI (Opus 4.6) ~6M tokens/85 turns, Code Gen (Opus 4.6) ~7M tokens/70 turns. Majority of cost from retransmission of previously seen content.

4. **Only ~20% of runs followed the same action sequence**: Agents discovered different valid UI paths while reaching the same goal state. This is both a strength (finds novel bugs) and a concern (non-deterministic results).

5. **Generated tests are fast once built**: ~3 minutes total (raw execution: ~32s thread reply, ~45s search discovery). The one-time generation cost becomes negligible when tests run repeatedly in CI.

## The Proposed Testing Pyramid

Four layers:
1. **Unit Tests** (bottom)
2. **Integration Tests**
3. **E2E Testing** — deterministic, fast, repeatable, CI-friendly, low cost
4. **Agentic Testing** (top, new layer) — goal-oriented, exploratory

Agentic testing is currently best suited for: exploring complex UI behavior, debugging flaky workflows, reproducing production bugs — not high-frequency CI.

## Strategic Summary

"Deterministic tests provide a stable foundation for CI, while agentic testing adds a distinct layer at the top of the testing pyramid for exploration, debugging, and validating complex behaviors."
