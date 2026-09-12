---
title: "What I learned building an opinionated and minimal coding agent"
url: https://mariozechner.at/posts/2025-11-30-pi-coding-agent/
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - agent-architecture
---

# What I Learned Building an Opinionated and Minimal Coding Agent

By Mario Zechner. Documents building "pi," a personal coding agent harness built from scratch: pi-ai (unified LLM API), pi-agent-core (agent orchestration), pi-tui (terminal UI), pi-coding-agent (CLI).

## Core Philosophy

"If I don't need it, it won't be built." Minimalism, control, observability, simplicity over feature richness.

## Key Technical Decisions

### Minimal System Prompt
Concise prompt explaining four core tools plus AGENTS.md injection. Frontier models understand coding agents through RL training; elaborate prompts add minimal value. Benchmark results support this.

### Four Essential Tools Only
- read: File contents with offset/limit
- write: Creates/overwrites files
- edit: Surgical text replacement
- bash: Synchronous command execution with timeouts

This minimalist toolset outperforms elaborate tool collections because models were trained on similar schemas.

### Full YOLO Mode by Default
No permission prompts, no safety rails. Existing "guardrails" merely create illusion of safety. If security matters, containerize -- don't add in-tool restrictions.

### Deliberate Omissions

- No built-in to-do lists: use TODO.md with checkboxes instead
- No plan mode: use PLAN.md for observability
- No MCP support: popular MCP servers consume 7-9% of context window before work begins. CLI tools with README files offer better token efficiency through progressive disclosure.
- No background bash: tmux provides superior visibility and co-debugging
- No sub-agents: invoke pi itself via bash within tmux for full observability

## On Context Engineering

Existing harnesses prevent proper context control through injected, unsurfaced data. Need APIs allowing explicit context specification and session serialization.

## On Security

Dual-LLM approaches and permission frameworks fail because models with read, execute, and network access cannot be solved through UI restrictions. Mirrors Simon Willison's conclusions about security theater.

## Benchmark Results

Terminal-Bench 2.0: pi competitive with Codex, Cursor, Windsurf. Terminal-Bench's own minimal agent (Terminus 2), which merely provides tmux sessions without sophisticated tooling, performs comparably -- suggesting elaborate tool abstractions may not confer meaningful advantages.

## Broader Implications

Feature accumulation creates complexity and API baggage. The success of tmux-based interaction and file-based state management suggests sophisticated abstractions within the agent tool layer often add complexity without corresponding benefit.

Pi demonstrates that competent coding agents require dramatically less scaffolding than contemporary harnesses provide.
