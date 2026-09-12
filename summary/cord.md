---
title: "Cord"
url: https://www.june.kim/cord
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - agent-orchestration
---

# Cord: Coordinating Trees of AI Agents

By June Kim. Proposes letting agents dynamically build task trees at runtime rather than requiring developers to predefine coordination structures.

## Problem with Existing Frameworks

- LangGraph: Static graphs requiring upfront developer decisions
- CrewAI: Developer-assigned roles prevent dynamic restructuring
- AutoGen: Unstructured conversation lacks dependency tracking
- OpenAI Swarm: Linear handoffs without parallelism
- Claude's Tool Loops: Single-threaded execution limits context

"Why are we still hardcoding the decomposition?" when models excel at natural planning.

## Innovation: Spawn vs. Fork

Two context-flow primitives:
- Spawn: Clean slate for independent subtasks; low overhead
- Fork: Sibling results injected; expensive but necessary for dependent analysis

Agents make runtime choices about how subtasks inherit context.

## Runtime Design

Built on Claude Code CLI with MCP tools backed by SQLite. ~500 lines of Python.

Five primitives: spawn(), fork(), ask(), complete(), read_tree() with dependency resolution.

## Behavioral Testing Insights

15 behavioral tests revealed emergent patterns: agents unprompted used read-verify cycles, escalation patterns emerged organically, error recovery through task respawning occurred naturally.

"The passing rate wasn't the insight -- it was that clear tool descriptions plus dependency semantics were sufficient for correct use."

## Protocol Independence

The five-primitive protocol is implementation-agnostic. Could extend to multiple LLMs, distributed systems (Postgres), human workers, or direct Claude API.

## Key Quote

"Real work isn't one task. It's a tree of tasks with dependencies, parallelism, and context that needs to flow between them."

"The protocol is the contribution. This repo is a proof of concept."
