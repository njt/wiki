---
url: https://github.com/xai-org/grok-build
title: "Grok Build"
author: SpaceXAI (xAI)
date_fetched: 2026-07-18
date_published: 2025
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

Grok Build (`grok`) is xAI's open-source terminal-based AI coding agent, written
entirely in Rust. It is a periodic snapshot from xAI's internal monorepo — source
is published but external contributions are not accepted.

The codebase is a Rust workspace spanning ~80 crates and well over 1M lines. The
largest crates are the TUI pager (425K lines), the agent shell/runtime (338K
lines), the tools crate (112K lines), and the workspace crate (78K lines). It
runs as a full-screen TUI, headlessly for scripting/CI, or embedded in editors
via the Agent Client Protocol (ACP).

Architecturally, the system is built around a `Tool` trait as its central
abstraction — every tool implements a streaming `execute` method that emits zero
or more `Progress` items followed by exactly one `Terminal` result. Tools are
registered in a `ToolBridge` central registry. Agents are assembled via a
10-step `AgentBuilder` fluent API, supporting allowlist/denylist resolution,
compat tool-name mapping (Claude's "Read/Bash/Grep/Edit" → Grok's ToolKind),
MCP tool integration, and skill discovery.

Key subsystems include: a transport-agnostic compaction engine with three
strategies (full-replace for coding agents, tail-keep and between-turn for
chat), an actor-based conversation state manager (`ChatStateActor`), a markdown-
plus-SQLite-vector memory system, a tree-sitter codebase graph with go-to-
definition and references, and a multi-layer config system supporting
Ed25519-signed enterprise policy. The hook system discovers JSON-defined hooks
from `~/.grok/hooks/` and executes them as child processes at session/tool
lifecycle events.

Notable design choices: TUI-first (unlike most coding agents which are CLI-
first), pure Rust end-to-end (avoiding GC pauses and enabling single-binary
distribution), a dedicated compaction model, and vendored Rust implementations
of Mermaid diagram parsing, layout, and SVG rendering — eliminating the Node.js
dependency common to other agents.
