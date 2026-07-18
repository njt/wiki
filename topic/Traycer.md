# Traycer

An open-source AI orchestration desktop app that wraps multiple coding agents (Claude Code, Codex, Cursor, OpenCode, and 12+ others) under a unified interface. Built as an Electron monorepo with a versioned, runtime-negotiated RPC protocol that enables the open-source client and closed-source agent runtime to ship independently. The protocol layer is the standout engineering artifact: every RPC method carries explicit upgrade/downgrade transforms verified at module load by JSON Schema fingerprint diffing. Agent-to-agent communication, real-time Yjs collaboration, and cross-provider context sharing make it a meta-harness — not another coding agent, but the infrastructure for orchestrating multiple agents simultaneously.

**Architecture**: The open-source repo contains the protocol (`@traycer/protocol`), CLI (`traycer-cli`), shared transport/auth, React GUI (`gui-app`), and Electron shell (`desktop`). The actual agent runtime is a separately shipped closed-source host binary provisioned from GitHub Releases. Communication between client and host uses a versioned RPC framework over WebSocket, with per-method `{major, minor}` schema versions negotiated at handshake (not npm semver). Each harness (17+ supported agents) normalizes its SDK-specific events into a shared `RuntimeEvent` discriminated union, enabling a single renderer to display any agent's output.

**Key insight**: The protocol's backwards-compatibility design is the project's real innovation. New features are folded onto existing method names to avoid breaking the handshake's equal-set check against already-shipped hosts. Downgrade paths between major versions carry explicit transforms — and when a downgrade is impossible (e.g., a v2.0 concept has no v1.0 representation), the error tells the caller exactly what to do. This is API lifecycle management as a type-checked, runtime-verified discipline, not documentation promises.

**Comparison**: Unlike [[Broomy]] (another multi-agent desktop app), Traycer provides real-time collaboration via Yjs CRDTs and a deeply architected protocol layer rather than a simpler side-by-side UI. Unlike [[Introducing Omnigent]] (Databricks' meta-harness), which wraps agents in a uniform API, Traycer is a full desktop application with its own renderer, not a library. Unlike [[Fleet Supervisor]] which focuses on parallel task execution, Traycer adds agent-to-agent debate and cross-model context switching within a single chat. The "harness matters more than the model" thesis — also central to [[Oh My Pi (omp)]] and [[Components of a Coding Agent]] — is operationalized here at scale.

#tool #project #agents #orchestration #open-source

---
*Sources: [[raw/traycer]]*
*Last updated: 2026-07-18*
