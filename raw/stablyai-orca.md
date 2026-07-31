---
url: https://github.com/stablyai/orca
title: Orca
author: stablyai
date_fetched: 2026-08-01
date_published: 2025
---

# Orca — Complete Repo Analysis

## Overview

Orca is a cross-platform Electron desktop application (macOS, Windows, Linux) that acts as an "AI orchestrator" — it runs multiple CLI AI coding agents (Claude Code, Codex, OpenCode, Pi, Grok, and 20+ others) in parallel git worktrees, with a mobile companion app (iOS/Android). It's a meta-tool: it doesn't implement AI models but provides the environment, coordination, and persistence layer for running existing CLI agents at scale.

**Scale**: ~590,000 lines of TypeScript across ~9,500 source files. Version 1.4.163-rc.1 at analysis time. MIT licensed.

## Architecture

### Process Architecture (Electron)

Orca uses Electron's standard multi-process model plus a custom daemon:

1. **Main process** (`src/main/`, ~888K lines Sloc): The application core. Manages app lifecycle, window creation, IPC handlers, PTY/provider system, agent hook server, SSH connections, orchestration engine, plugin system, persistent storage, telemetry, updates, and crash recovery. Entry point: `src/main/index.ts` (3,034 lines).

2. **Renderer process** (`src/renderer/`, ~22K lines): React 19 + Tailwind CSS 4 web UI with shadcn/ui component library, Monaco editor, Tiptap rich text, xterm.js terminals with WebGL rendering, dnd-kit drag-drop, TipTap markdown editor, Mermaid diagrams, KaTeX math, PDF.js. Single-page app loaded via `src/renderer/index.html`.

3. **Preload layer** (`src/preload/`, ~10K lines): Electron preload scripts bridging main ↔ renderer via contextBridge/ipcRenderer.

4. **CLI** (`src/cli/`, ~32K lines): Standalone `orca` CLI binary for scripting and agent-driven control. Handlers for worktree, browser, computer use, emulator, Linear, account management, etc.

5. **Relay** (`src/relay/`, ~48K lines): A long-lived relay process that maintains WebSocket connections for mobile pairing and SSH tunnel maintenance. Contains the relay HTTP client, session broker, E2E encryption, auth coordinator, and demand ledger for metering mobile ↔ desktop traffic.

6. **Daemon** (`src/main/daemon/`): A forked child process (via `child_process.fork`) that hosts persistent PTY sessions so they survive renderer crashes and window close. Communicates with main via IPC.

7. **Mobile app** (`mobile/`): React Native companion app (iOS/Android) for monitoring and steering agents from a phone.

### Key Architectural Patterns

**Persistent PTY via Daemon**: Instead of hosting PTY sessions in the main process (which would die on quit), Orca spawns a forked daemon process that owns the actual `node-pty` pseudo-terminals. The main process routes data between the daemon and the renderer. On restart, the main process reconnects to surviving daemon PTYs, and the renderer rehydrates terminal state from persistence. This is the single most architecturally significant decision — it's what makes terminal sessions survive renderer crashes and window close.

**Agent Hook Server**: A local HTTP server (`src/main/agent-hooks/server.ts`) that listens on a loopback port for status pings from agent CLI hooks. Each supported agent (Claude Code, Codex, Pi, Grok, Cursor, etc.) has a managed hook installed into its runtime that POSTs state changes (idle → working → waiting → blocked → done) to this server. The server enriches events with timing, persists last-known state to disk (`last-status.json`), and forwards to the renderer via IPC. This is Orca's primary mechanism for knowing what agents are doing without parsing terminal output.

**Hook installation is per-agent**: For Claude Code it's via `.claude/hooks/`, for Codex via `.codex/hooks/`, for Pi via its native hook protocol, etc. The `agent-hooks/` directory contains agent-specific installers that know each agent's hook contract. For remote/SSH agents, hooks are relayed through the SSH multiplexer back to the local hook server.

**Worktree Isolation**: Every agent session gets its own git worktree (or folder workspace). The worktree system (`src/main/git/worktree.ts`, `src/main/worktree-create-base.ts`) handles creation, base resolution, stale detection, and teardown. Worktrees can be local, WSL, or SSH-remote. Orca tracks per-worktree metadata (display name, first-agent-message rename, creation source) in a JSON store.

**OrcaRuntimeService** (`src/main/runtime/orca-runtime.ts`): The central orchestrator singleton. It manages:
- The live graph of worktrees, terminals, browser panes, and emulator panes
- Agent session claim identity (cryptographic session binding)
- PTY handle tracking and synthetic title injection
- Worktree lifecycle (create, clone, delete, rename)
- Orchestration dispatch (coordinator → worker fan-out)
- File watcher registration and SSH re-arm
- Mobile session tab synchronization

**SSH Multiplexer** (`src/main/ssh/`): A custom SSH connection manager built on top of `ssh2`. Key features:
- Single SSH connection multiplexed across PTY, SFTP, git, file watching, and agent hook channels
- Auto-reconnect with incarnation tracking
- Port forwarding provider
- SSH config file parsing (OpenSSH-compatible)
- WSL-aware SSH (bridges Windows ↔ WSL SSH)
- Relay deployment: auto-installs Orca runtime on remote hosts
- Cross-platform: native, WSL, and SSH execution hosts

**Relay System** (`src/main/runtime/relay/`): Desktop ↔ mobile synchronization via a relay broker. Uses:
- WebSocket transport with E2E encryption (tweetnacl)
- Session broker for auth and routing
- Demand ledger for metering
- Mobile-specific RPC methods (terminal subscribe, file search, notification replay)

### Plugin System

`src/main/plugins/` implements a plugin architecture with:
- Plugin discovery from filesystem directories
- Bundled bootstrap plugins shipped with the app
- Marketplace integration for community plugins
- Consent and enablement management
- Kill list service for blocking known-malicious plugins
- Plugin-hosted in a separate child process for isolation

### Storage and Persistence

Orca uses a custom JSON store (`src/main/persistence.ts`) for all settings and metadata. There's no SQLite for core state — only JSON files on disk. The orchestration system uses a separate SQLite database (`src/main/runtime/orchestration/db.ts`) for coordinator runs, task dispatch, and message history.

Terminal history is managed by `src/main/terminal-history.ts` with garbage collection, async deletion, and tombstone retry.

### Browser Integration

`src/main/browser/` wraps Chromium (via Electron's BrowserView/webview) with:
- Agent-controllable browser panes (Design Mode for element picking)
- Offscreen browser backend for headless operation
- Certificate trust management
- Agent Browser Bridge (browser ↔ agent communication)

### Computer Use

`src/main/computer/` provides desktop automation via native accessibility APIs (macOS: Accessibility framework, Windows: UIA) for agents that need to operate desktop applications.

### Supported Agent Integrations

Each agent gets its own integration module:
- `src/main/claude/` — Claude Code (hook service, trust presets, session resume, live PTY persistence)
- `src/main/codex/` — Codex (hook service, runtime home management, session resume/migration, real-home vs managed-home)
- `src/main/pi/` — Pi
- `src/main/grok/` — Grok
- `src/main/cursor/` — Cursor
- `src/main/copilot/` — GitHub Copilot
- `src/main/opencode/` — OpenCode
- `src/main/droid/` — Droid
- `src/main/gemini/` — Gemini CLI
- `src/main/antigravity/` — Antigravity
- `src/main/kimi/` — Kimi
- `src/main/mimo/` — MiMo Code
- `src/main/devin/` — Devin
- `src/main/hermes/` — Hermes Agent
- `src/main/command-code/` — Command Code
- `src/main/minimax/` — MiniMax
- `src/main/amp/` — Amp
- `src/main/openclaude/` — OpenClaude

Each follows a similar pattern: account service, runtime auth/selection, hook installation, and session resume.

## Key Techniques

### 1. Agent Status via OSC Title + Hook Server Dual Path

Orca detects agent state through two channels:
- **OSC terminal titles** (`\x1b]0;...\x07`): Parsed from PTY output by `src/shared/agent-detection.ts` and tracked per-terminal in `orca-runtime.ts`. This is the fallback for agents without managed hooks.
- **Hook server pings**: The primary mechanism. Hooks POST JSON payloads directly, which is more reliable and carries structured data (agent type, state, model, session metadata).

The hook server enriches events with `receivedAt` and `stateStartedAt` timestamps, persists them to `last-status.json`, and forwards to the renderer. The two paths are unified in `src/shared/agent-hook-listener.ts` which normalizes payloads from both sources.

### 2. Synthetic Title Injection

For agents that don't natively emit OSC titles during active work, Orca injects synthetic title frames directly into the PTY data stream via `sendSyntheticTitle()`. This drives the UI spinner/status indicators. The injection is gated on window visibility to avoid unnecessary work. A shared interval timer (80ms) advances all active spinners, avoiding per-pane timers.

### 3. Persistent PTY via Forked Daemon

Rather than hosting PTY sessions in the main process, Orca forks a daemon child process that owns `node-pty` pseudo-terminals. The main process sends commands and receives data via IPC. This means:
- Terminal sessions survive renderer crashes
- Terminal sessions survive window close (hidden to tray)
- Terminal sessions survive main process restart (daemon checkpoint/restore)

The daemon uses `node-pty` patched (see `config/patches/node-pty@1.1.0.patch`) for custom behavior.

### 4. SSH Connection Multiplexing Over a Single TCP Connection

Instead of opening separate SSH connections for PTY, SFTP, file watching, and git, Orca multiplexes all channels over one SSH connection using `ssh2`'s session/channel APIs. The `ssh-channel-multiplexer.ts` handles:
- Concurrent channel allocation
- Backpressure management
- Settlement (waiting for channel open confirmation)
- Lane scheduling for fair writer allocation

### 5. Codex Session Resume Architecture

The Codex integration has the most complex session resume logic in the codebase. Orca intercepts the Codex CLI launch to inject `--resume <session-id>` when a prior session exists. It handles:
- Session provenance tracking (which CODEX_HOME owns which session)
- Legacy session migration from Orca-managed to system `~/.codex`
- Real-home vs. managed-home hook installation
- Project trust pre-marking before PTY spawn

### 6. Worktree Staleness Detection in Orchestration

The orchestration coordinator (`src/main/runtime/orchestration/coordinator.ts`) implements a stale-base detection system. Before dispatching a task to a worker agent, it probes the worktree's git drift (commits behind base). If behind more than 20 commits (`DISPATCH_STALE_THRESHOLD`), it escalates rather than dispatching. Spec authors can opt out with `allow-stale-base: true` in the spec text.

### 7. Crash Recovery Architecture

Orca has extensive crash recovery mechanisms:
- **Renderer crash recovery**: Automatic reload with exponential-ish backoff, circuit breaker after N consecutive crashes, manual recovery prompt as last resort
- **GPU crash fallback**: Detects GPU process death bursts, prompts user to restart with software rendering, persists fallback marker across launches
- **Main thread hang watchdog**: Detects event loop stalls, writes hang detection marker for next launch to read
- **Crash breadcrumbs**: Coalesced crash breadcrumb store (last 30 events) included in crash reports, with durable (disk-persisted) breadcrumbs for critical transitions
- **Process-gone classification**: Distinguishes expected teardowns (app quit, renderer reload) from unexpected crashes to avoid false crash reports

### 8. Agent Prompt Injection via Bracketed Paste

`src/shared/agent-prompt-injection.ts` implements bracketed paste mode injection for submitting prompts to agent terminals. This avoids line-by-line typing which would be slow and trigger agent hooks mid-paste. The system builds paste bytes with appropriate delays (`AGENT_PROMPT_SUBMIT_DELAY_MS`) and handles oversized inputs through chunked iteration.

### 9. LLM-Friendly CLI Output Format

The CLI supports multiple output formats (`--format json`, `--format text`) designed for both human and agent consumption. The `src/cli/format.ts` module normalizes output across handlers with structured error reporting.

## Design Decisions

### Optimized For: Multi-Agent Parallelism

The core value proposition is running N agents in parallel, each in its own worktree. Everything flows from this: PTY persistence, hook-based status tracking, worktree isolation, SSH multiplexing. The architecture sacrifices simplicity for parallelism support.

### Optimized For: Persistence and Crash Recovery

Enormous engineering effort went into making things survive: daemon-based PTYs, crash breadcrumbs, renderer recovery, GPU fallback, update handoff, agent session resume across restarts. This is the most distinguishing architectural choice — most Electron apps lose terminal state on crash; Orca treats that as a design constraint to solve.

### Sacrificed: Simplicity

At 590K lines with 9,500+ files, this is a complex codebase. The main process entry point is 3,000+ lines. Many subsystems have deep coupling: agent hook server ↔ runtime ↔ PTY provider ↔ daemon ↔ renderer. The per-agent integration modules are individually complex (Codex alone spans multiple modules for accounts, hooks, sessions, trust, usage tracking).

### Sacrificed: Test Coverage for Orchestration Logic

While there are many unit tests (`.test.ts` files throughout), the orchestration coordinator's core loop (decompose → dispatch → monitor → converge) appears to have limited integration testing. The coordinator depends on live terminals and git operations that are hard to mock realistically.

### Trade-off: JSON Store vs. SQLite for Core State

The app uses a JSON file store for settings, worktree metadata, and workspace session state. This is simple and debuggable but doesn't scale well for concurrent writes or large datasets. The orchestration system uses SQLite separately — a split that suggests the JSON store was an early choice that became hard to change.

### Trade-off: Electron for Desktop + React Native for Mobile

Using Electron gives cross-platform reach and access to Chromium for browser panes, but at significant resource cost. The React Native mobile app is a pragmatic companion rather than a full-featured client — it monitors and steers, but doesn't run agents.

## Comparison Notes

### vs. Traycer

Traycer is another open-source AI orchestration desktop app. Both wrap multiple coding agents in a unified interface. Key differences:
- Traycer uses Yjs for real-time collaboration; Orca uses a custom relay protocol
- Traycer's RPC protocol is versioned for independent client/host releases; Orca's IPC is more tightly coupled
- Orca has a mobile companion app; Traycer is desktop-only
- Orca supports 20+ agents with per-agent hook integration; Traycer's agent list is smaller (17+)

### vs. Broomy

Broomy is the closest comparison — another MIT-licensed Electron app for running multiple coding agents side-by-side. Orca is significantly larger in scope: SSH remote worktrees, mobile companion, orchestration engine, computer use, plugin system. Broomy is more focused on the side-by-side editing experience.

### vs. cmux

cmux is a macOS-native terminal built on libghostty, designed for managing multiple AI coding agent sessions. Both address the "parallel agent" use case. cmux is simpler (native, no Electron) with notification rings for attention; Orca is the full platform with orchestration, SSH, mobile, and more.

### vs. Fleet Supervisor

Fleet Supervisor (sermakarevich) is a Python supervisor for parallel coding agents with pluggable backends and a web UI. Orca's orchestration is more deeply integrated (worktree management, SSH, PTY persistence) but Fleet Supervisor has the advantage of being backend-agnostic and having a built-in MCP question broker.

### vs. Grok Build

Grok Build is a pure Rust terminal-based coding agent — it IS an agent, not an orchestrator of agents. The comparison point is that both are open-source, terminal-first tools. Orca orchestrates existing agents; Grok Build implements its own agent from scratch. Orca supports Grok as one of its 20+ launchable agents.

## File Listing (Key Modules)

```
src/main/
  index.ts (3034 lines) — App entry, lifecycle, service wiring
  persistence.ts — JSON store singleton
  runtime/
    orca-runtime.ts — Central orchestrator singleton
    orchestration/
      coordinator.ts — Task decomposition and dispatch loop
      db.ts — SQLite-backed orchestration database
      types.ts — Message/Task/Run types
      federation-sync.ts — Cross-worktree dispatch sync
    relay/
      desktop-relay-service.ts — Mobile pairing relay
      relay-session-broker.ts — Auth and routing
    runtime-rpc.ts — WebSocket RPC server
  agent-hooks/
    server.ts — Local HTTP hook receiver
    managed-agent-hook-controls.ts — Hook enable/disable
    first-work-branch-rename.ts — Auto-rename on first agent message
  pty/
    (in providers/) — local-pty-provider.ts, daemon-init.ts
  ssh/
    ssh-connection.ts — SSH2 wrapper with multiplexing
    ssh-channel-multiplexer.ts — Multi-channel over single connection
    ssh-relay-session.ts — Remote agent hook relay
    ssh-config-parser.ts — OpenSSH config parser
    ssh-relay-deploy.ts — Auto-deploy Orca runtime to remote
  git/
    worktree.ts — Git worktree operations
    runner.ts — Git binary execution wrapper
  browser/
    agent-browser-bridge.ts — Agent ↔ browser communication
    browser-manager.ts — Browser pane lifecycle
  claude/ — Claude Code integration
  codex/ — Codex CLI integration
  plugins/ — Plugin system
  terminal-history.ts — Scrollback persistence
  startup/ — Process startup, GPU, single-instance lock
  crash-reporting/ — Crash recovery and reporting
src/cli/
  dispatch.ts — CLI command router
  handlers/ — Per-command handler modules
  format.ts — Output formatting (JSON, text)
src/renderer/src/ — React UI
src/shared/ — Cross-process types, detection, protocols
src/relay/ — Relay process
mobile/ — React Native companion app
```

## Technical Debt Indicators

1. **main/index.ts at 3,034 lines**: The entry point has `max-lines` disabled with explicit rationale, but the sheer number of imports and service wiring suggests a lack of modular decomposition at the top level.

2. **orca-runtime.ts at hundreds of lines**: Also has `max-lines` disabled — "still owns the mutable live graph, PTY handles, waiters, mobile floor/layout state, and managed-worktree reconciliation."

3. **agent-hooks/server.ts**: Similarly `max-lines` disabled — "owns the loopback HTTP adapter, the on-disk last-status persistence layer, and the relay ingest path."

4. **Per-agent code duplication**: Each agent integration (claude, codex, grok, cursor, etc.) follows similar patterns (account service, hook service, session resume, trust) with duplication rather than shared abstractions.

5. **Two persistence backends**: JSON files for core state, SQLite for orchestration — suggesting organic growth rather than architectural intent.
