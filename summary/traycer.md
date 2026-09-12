---
topics:
  - agent-architecture
---
# Traycer (Summary)

Traycer is an open-source AI orchestration desktop application (Electron + React) that wraps 17+ coding agents (Claude Code, Codex, Cursor, OpenCode, and others) under a unified interface with real-time collaboration via Yjs CRDTs. The open-source repo contains the client UI, CLI, and a sophisticated versioned RPC protocol layer; the actual agent runtime is a closed-source host binary provisioned separately.

The protocol layer is the standout engineering artifact. Every RPC method carries `{major, minor}` schema versions with explicit upgrade/downgrade transforms verified at module load by JSON Schema fingerprint diffing. Minors must be additive; majors must contain at least one breaking change. A "released floor" of 118 method names defines the minimum set all peers must support, with graceful degradation for newer methods.

Agent-to-agent communication is fire-and-forget (`agent.sendMessage`); child agents are created via `agent.create` with four profile selection modes. Each harness normalizes its SDK-specific events into a shared `RuntimeEvent` discriminated union (~50 event types covering text, reasoning, tool calls, file changes, subagents, plans, todos, compactions, workflows, and provider notices).

The key design insight: backwards compatibility is a first-class, type-checked, runtime-verified discipline — not documentation promises. This enables the open-source client and closed-source host to ship asynchronously, with any version talking to any version within compatible ranges.

*Source: [[raw/traycer]]*
