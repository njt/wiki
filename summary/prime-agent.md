---
url: https://github.com/PrimeIntellect-ai/prime-agent
title: "Prime Agent: A Self-Improving RLM Agent"
author: Prime Intellect
date_fetched: 2026-08-21
topics:
  - agent-architecture
---

# Prime Agent: A Self-Improving RLM Agent

Prime Intellect's open-source coding and research agent for general and long-running work, built around two abstractions. The **Recursive Language Model (RLM)** treats context as a variable and tools like sub-agents as function calls inside a persistent IPython REPL — the model programs its own tool use in Python rather than emitting fixed tool-call schemas. The **Continual Harness** stores prompts, memories, skill descriptions, and reusable sub-agent specs as durable state the agent can refine through small, evidence-backed edits via `/refine`.

The repo is a fork of Pi: the `packages/coding-agent` workspace is literally `@earendil-works/pi-coding-agent` (v0.7.4), and Prime Agent layers its novelty on top of Pi's agent loop. The monorepo spans four TypeScript workspaces (`ai` for the provider layer, `agent` for the loop, `tui` for the terminal, `coding-agent` for the CLI) plus `prime-agent-runtime`, a Python package preloaded into every IPython kernel.

Execution is split across three processes: a client (TUI or headless print/JSON/RPC) owns rendering; a daemon supervisor owns discovery, routing, and cross-agent message delivery; and a session worker owns one root session, its scheduler, and the IPython kernel. Workers and kernels are separate processes for lifecycle and failure containment — not security sandboxes. Sessions persist as append-only JSONL, and the kernel's Python namespace survives resume via a per-variable `dill` snapshot.

Key surface features: sub-agents spawn with `await rlm("task")` and return a handle immediately (results arrive via `agent_message`), sessions keep running in a background daemon and can be reattached, `/refine` never rewrites the immutable base system prompt, skills are importable Python packages, and automatic compaction plus persistent goals, heartbeats, and bounded autonomous mode keep long tasks moving.
