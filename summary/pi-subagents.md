---
url: https://github.com/nicobailon/pi-subagents
title: pi-subagents
author: Nico Bailon
date_fetched: 2026-08-07
topics:
  - agent-orchestration
---

# pi-subagents

`pi-subagents` is a TypeScript extension for the Pi coding agent that adds a multi-agent delegation system: a single `subagent` tool with TypeBox-validated parameters for spawning focused child Pi sessions. It ships with six built-in agents (scout, researcher, worker, reviewer, oracle, delegate), a chain execution engine for sequential→parallel→dynamic fan-out workflows, a sandboxed JavaScript workflow DSL running in Node.js worker threads, an RPC protocol for external control, background/async execution with a FleetView TUI, an opt-in watchdog adversarial reviewer, durable mission tracking, intercom bridge for parent-child communication, and external CLI runner support. Version 0.42.1, MIT licensed, installs via `pi install npm:pi-subagents`.

## How it works

Pi is the parent session. A subagent is a focused child Pi session with its own job, model, tool budget, and capability ceiling. When a user asks for a subagent, Pi calls the `subagent` tool with parameters specifying the agent type, task, and execution mode. Foreground runs stream progress inline; background runs detach and can be monitored via FleetView.

The extension hooks into Pi's lifecycle events (session_start, session_shutdown, agent_start, agent_end, session_compact, tool_result) and registers message renderers for slash results, subagent notifications, steering notices, and control notices. It manages spawn budgets, capability ceilings, and result watcher polling.

## Built-in agents

Six agents ship as markdown files with YAML frontmatter defining name, description, tools, model, thinking level, and skills:

- **scout** — Fast codebase recon with low thinking, read-only tools, outputs context.md
- **researcher** — Web/docs research with sources and a concise brief
- **worker** — Implementation with strict tool allowlist, escalates unapproved decisions
- **reviewer** — Five review types (diffs, plans, solutions, codebase health, PR/issues)
- **oracle** — Second opinion before acting, challenges assumptions without editing
- **delegate** — Lightweight general delegate behaving close to the parent session

Agents can be overridden or extended via user/project directories and config files. External CLI runners (non-Pi subprocesses) are also supported.

## Execution modes

**Chains**: Sequential steps → parallel steps → dynamic fan-out (expand from prior structured output via JSON Pointer, fan out N children, collect results). Supports concurrency limits, fail-fast, checkpoints, acceptance gates, and worktree isolation.

**Workflow scripts**: Sandboxed JavaScript DSL running in Node.js worker threads. Provides `runs` object with run(), all(), status(), ref(), refs() methods and `state` with get()/set() for mission workflows. Validates keys against patterns, detects duplicate keys with incompatible params.

**Foreground vs. async**: Foreground runs stream in the conversation; async runs detach and are polled via result watcher. The FleetView TUI below the editor keeps active work visible. `/subagents-fleet` opens a live inspector.

## Safety mechanisms

**Watchdog**: Opt-in adversarial change reviewer running a separate model that inspects turn deltas for issues. Scope monitoring, LSP checks, and child tool permissions.

**Capability ceilings**: Child agents can be restricted below parent capabilities — limited tools, models, turn budgets, tool budgets, usage budgets.

**Acceptance gates**: Three levels — attested (agent self-reports), checked (supervisor validates), verified (evidence required, supervisor adjudicates).

**Worktree isolation**: Agents can run in isolated git worktrees so parallel mutations don't conflict, matching the pattern from Claude Code's dynamic workflows.

## Extension API

RPC protocol v1 over events with seven methods: ping, status, spawn, steer, interrupt, stop, resume. Delegation API for extension-to-extension communication. Preflight hooks for agent launch interception. Background-work provider interface. Herdr inspector integration.

## Key facts

- 233K+ lines of TypeScript across the repo
- Single `index.ts` re-export entry point
- TypeBox schemas for all tool parameters (~376 lines)
- RPC ~654 lines with fleet status aggregation (max 16 entries, 256 candidates)
- Agent discovery: builtins + user dirs (~/.agents, ~/.pi/agent/agents) + project dirs (.agents/)
- Chain execution ~1,527 lines: sequential, parallel, dynamic fan-out
- Workflow sandbox: Node.js worker threads with validated keys and gate support
- Dependency chain: @earendil-works/pi-agent-core, pi-ai, pi-coding-agent, pi-tui
