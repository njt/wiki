# Orca

An open-source Electron desktop app (macOS/Windows/Linux) that orchestrates 20+ CLI AI coding agents — Claude Code, Codex, Pi, Grok, Cursor, and others — in parallel git worktrees, with a mobile companion app for monitoring and steering from a phone. ~590K lines of TypeScript. MIT licensed by stablyai.

---

## Architecture

Orca is a **multi-agent orchestrator**, not an agent itself. It wraps existing CLI coding agents in persistent terminal sessions, each isolated in its own git worktree or folder workspace, and provides coordination, persistence, and remote access layers.

### Process model

1. **Main process** (`src/main/index.ts`, 3,034 lines) — Electron main, wires all services
2. **Renderer** (`src/renderer/`) — React 19 + Tailwind + shadcn/ui + Monaco + xterm.js WebGL
3. **Daemon** (forked child) — hosts persistent `node-pty` sessions so terminals survive renderer crashes and window close
4. **Relay** (`src/relay/`) — long-lived WS process for mobile pairing and SSH tunnel maintenance
5. **CLI** (`src/cli/`) — standalone `orca` binary for scripting and agent-driven control
6. **Mobile** (`mobile/`) — React Native companion (iOS App Store, Android APK)

### The daemon is the key architectural decision

Rather than hosting PTY sessions in the main process (which would die on quit), Orca forks a daemon that owns the actual pseudo-terminals. The main process routes data between daemon and renderer. On restart, the main process reconnects to surviving daemon PTYs and the renderer rehydrates terminal state from persistence. This single decision makes terminal sessions durable across crashes — most Electron apps lose terminal state on crash; Orca treats that as a design constraint to solve.

### Agent status via dual-path detection

Orca knows what agents are doing through two channels:

- **Hook server** (primary): A local HTTP server (`src/main/agent-hooks/server.ts`) receives structured status pings from managed hooks installed into each agent's runtime. State transitions (idle → working → waiting → blocked → done) flow directly via JSON POSTs, enriched with timestamps, persisted to `last-status.json`, and forwarded to the renderer via IPC.

- **OSC terminal title parsing** (fallback): For agents without managed hooks, Orca parses `\x1b]0;...\x07` escape sequences from PTY output (`src/shared/agent-detection.ts`).

For agents that don't emit visible working-state indicators, Orca injects **synthetic title frames** directly into the PTY stream — a shared 80ms interval timer drives Unicode spinner characters that appear as OSC titles in the renderer.

### SSH multiplexing, not per-channel connections

Instead of opening separate SSH connections for PTY, SFTP, file watching, and git, Orca multiplexes all channels over one `ssh2` connection (`src/main/ssh/ssh-channel-multiplexer.ts`) with backpressure management, lane scheduling, and incarnation tracking for reconnect. The SSH subsystem also handles auto-deploy of the Orca runtime to remote hosts and OpenSSH config file parsing.

### Orchestration engine

`src/main/runtime/orchestration/coordinator.ts` implements a decompose → dispatch → monitor → converge loop backed by SQLite. Key features:
- **Stale base detection**: Before dispatching, probes worktree git drift; if >20 commits behind base, escalates rather than dispatching (opt-out via `allow-stale-base: true` in spec)
- **Federation**: Cross-worktree dispatch sync via `federation-sync.ts`
- **Lifecycle reconciliation**: Detects and reconciles split-brain states between coordinator and workers
- **Legacy worker terminal recovery**: Survives and recovers worker terminals from prior sessions

### Plugin system

`src/main/plugins/` supports filesystem-discovered plugins, bundled bootstrap plugins, a marketplace, consent/enablement management, and a kill list for known-malicious plugins. Plugins run in an isolated child process.

## Key Techniques

### Persistent PTY via forked daemon

The daemon child process (`src/main/daemon/daemon-init.ts`) owns the `node-pty` instances. The main process sends commands and receives data via IPC. On quit, the daemon checkpoints state. On restart, the main process reconnects. This is the most distinguishing technical choice — terminal scrollback, agent state, and running processes survive restart.

### Per-agent hook installation

Each of the 20+ supported agents gets its own hook installer in `src/main/agent-hooks/`. For Claude Code it's via `.claude/hooks/`, for Codex via `.codex/hooks/`, for Pi via its native hook protocol. Remote agents relay hooks through the SSH multiplexer back to the local hook server. WSL agents use a WSL hook relay manager that bridges Windows ↔ Linux filesystem boundaries.

### Codex session resume — the most complex integration

The Codex integration (`src/main/codex/`) handles session provenance tracking across real `~/.codex` and Orca-managed homes, legacy session migration, project trust pre-marking, and hook installation that varies by whether the user selected system-default or managed-home runtime. The launch prep function is called before every Codex PTY spawn and must be synchronous.

### Synthetic title spinners

When an agent is in `working` state, Orca injects Unicode spinner frames (`⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏`) at 80ms intervals into the PTY data stream via OSC title escapes. One shared timer drives all spinners (avoiding per-pane interval timers). The spinner stops when the hook server reports a state change to `blocked`, `waiting`, or `done`. Visibility-gating avoids unnecessary work when the window is hidden.

### Crash recovery layers

- **Renderer crash**: Auto-reload with backoff, circuit breaker after N consecutive crashes, manual recovery prompt
- **GPU crash fallback**: Detects GPU process death bursts, offers restart with `--disable-gpu`, persists marker across launches
- **Main thread hang watchdog**: Detects event loop stalls, writes marker for next launch
- **Crash breadcrumbs**: Coalesced store (last 30 events) + durable disk-persisted breadcrumbs for critical transitions
- **Process-gone classification**: Distinguishes expected teardowns from real crashes

### Agent prompt injection via bracketed paste

`src/shared/agent-prompt-injection.ts` builds paste-mode byte sequences for submitting prompts to agent terminals, avoiding line-by-line typing that would trigger hooks mid-paste. Includes delay management and oversized-input chunking.

## Design Decisions

**Optimized for**: Running many agents in parallel, each in its own worktree. Everything flows from this: daemon-based PTYs, hook-based status, SSH multiplexing, orchestration dispatch.

**Optimized for**: Crash survival. Daemon PTYs, breadcrumbs, renderer recovery, GPU fallback, update handoff, session resume — enormous engineering investment in making things not break.

**Sacrificed**: Architectural simplicity. 590K lines, 9,500+ files, 3,000-line entry point, deep coupling between subsystems. The per-agent integration modules follow similar patterns with significant duplication.

**Trade-off**: JSON file store for core state vs. SQLite for orchestration — two persistence backends that grew organically rather than by design.

**Trade-off**: Electron's resource cost in exchange for Chromium's browser pane support (Design Mode, offscreen browser) and cross-platform reach.

## Comparison Notes

- **vs. [[Traycer]]**: Both open-source Electron orchestrators wrapping multiple agents. Traycer uses Yjs for real-time collaboration; Orca uses a custom relay protocol and adds mobile companion + SSH worktrees + computer use.
- **vs. [[Broomy]]**: Both MIT-licensed Electron apps for parallel agent sessions. Broomy is focused on side-by-side editing; Orca is the full platform with orchestration, SSH, mobile, plugins.
- **vs. [[cmux]]**: cmux is a native macOS terminal for multi-agent management; Orca is the cross-platform Electron equivalent with orchestration and remote access.
- **vs. [[Fleet Supervisor (sermakarevich)]]**: Python supervisor with pluggable backends. Orca's orchestration is more deeply integrated with the OS (PTY, git, SSH) but Fleet Supervisor's backend-agnostic design is more composable.
- **vs. [[Grok Build]]**: Grok Build IS an agent; Orca orchestrates agents (including Grok). Complementary, not competing.
- **vs. [[Loop Engineering]]**: Orca is Addy Osmani's concept of "designing systems that prompt agents" made concrete — a desktop app implementing automations + worktrees + skills + connectors + state management as a product.
- **vs. [[Xirp]]**: Spotify's macOS parallel-agent control plane — same category (persistent terminals + worktree-per-task + one control surface), but Xirp's differentiator is the Spotify Portal context layer (Backstage Software Catalog + Workspaces injected over MCP), not orchestration breadth. Xirp is enterprise-context-first and narrow (macOS, three agents, beta); Orca is breadth-first (20+ agents, SSH, mobile, plugins) with no organizational context layer.

---

*Sources: [[raw/stablyai-orca]]*
*Last updated: 2026-08-01*
