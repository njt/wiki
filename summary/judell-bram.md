---
url: https://github.com/judell/bram
title: Bram — AI-Assisted Software Development Desktop Shell
author: Jon Udell
date_fetched: 2026-06-22
date_published: 2026-06-02
topics:
  - agent-coding-workflow
  - claude-code
---

# Bram — Full Architectural Analysis

## Project Overview

Bram is a Tauri desktop application (v0.2.12) that wraps AI coding agents (Claude Code and Codex CLI) in a structured workflow shell. It provides a real terminal alongside an agent tools pane, with an optional target-app iframe for preview. The core thesis: git and GitHub are already the right tools for versioning, collaboration, and accountability — Bram guides agents to use them well rather than replacing them.

Created by Jon Udell, discussed in his blog posts "How to make best use of git and GitHub for AI-assisted software development" (2026-06-02) and "Vibe Coding as a Team Sport" (2026-06-17).

## File Tree & Project Scale

- **24,439 lines** — `src-tauri/src/lib.rs` (Rust backend)
- **1,600 lines** — `app/main.js` (parent shell frontend)
- **4,385 lines** — `app/__shell/helpers.js` (shell helpers / XMLUI bridge)
- **1,089 lines** — `app/__shell/conventions.md` (agent conventions, loaded by CLAUDE.md @-import)
- **604 lines** — `app/__shell/worklist-guard.py` (Claude Code PreToolUse hook)
- **517 lines** — `docs/apis.md` (API catalog: HTTP routes, IPC commands, filesystem coordination)
- **213 lines** — `docs/spine.md` (turn-state spine architecture)
- **19 XMLUI files** — Agent pane UI components
- **3 Rust files** — main.rs, lib.rs, build.rs
- **6 Python files** — worklist guards and scripts
- **0 lines of TypeScript**, no bundler, no package.json — frontend is static vanilla JS

## Architecture

### Tech Stack
- **Desktop shell**: Tauri 2 (Rust + platform webview)
- **Backend**: Rust (24,439-line lib.rs monolith)
- **Terminal**: xterm.js + portable-pty (Rust crate), PTY → WebView via Tauri Channel (binary chunks as Uint8Array)
- **Agent pane**: XMLUI (custom reactive UI framework from xmlui.org), served from embedded `app/tools/` tree
- **Frontend**: Vanilla JS, no framework, no bundler, no package.json
- **No database** — all state lives in files under `resources/` watched by the Rust backend

### Three-Pane Layout
1. **Terminal** (left) — xterm.js with PTY backend, connects to bash/powershell running Claude Code or Codex CLI
2. **Target app** (right, optional, off by default) — iframe proxying the project's dev server through `tauri://localhost/__project/*`
3. **Agent tools drawer** (bottom-right) — iframe serving `app/tools/index.html`, XMLUI app with tabs: Worklist, Commits, Issues, Sessions, History, Context, Status, Settings

### Transport Architecture (per docs/apis.md)
Bram uses five transport layers between the host (Rust) and consumers (iframes, agents):

1. **Tauri scheme routing** — `tauri://localhost/__*` URLs for browser-facing iframes
2. **Internal HTTP-style routes** — `/__worklist`, `/__commits`, `/__sessions/*`, etc., served by tiny_http in the Rust process
3. **Loopback HTTP** — `http://127.0.0.1:<bram-port>/...` for Claude's `curl` calls (read from `resources/.bram-port`)
4. **Tauri IPC** — `window.__TAURI__.core.invoke()` for direct WebView↔host commands (pty_write, git_push, whisper_start)
5. **Filesystem coordination** — `resources/.worklist-intent.json` / `.worklist-result.json` for Codex (which can't reach loopback due to sandbox), plus `.worklist-authorization.json`, `.inflight-claim.json`

### Key Rust Structures (lib.rs)
- `PtyState` — wraps `Box<dyn MasterPty + Send>` + `Box<dyn Write + Send>`
- `PaneUrls` — holds right_pane, tools, default_right_pane, right_pane_upstream, loopback_origin URLs
- `ProjectConfig` — deserialized from `.bram.json` (server, shell, worklist, ui, traces, menus blocks)
- `WorklistAuthorizationRecord` — `{kind, ids, items, mismatched_ids, issued_at_ms, source, consumed_at_ms}`
- `ActiveProjectState` — Mutex-wrapped PathBuf for the project root
- `SpawnedServerState` — optional project dev server child process
- `WhisperState` — optional whisper-server child process

### Embedded App Bundle
The `app/` tree is compiled into the binary via `include_dir!` macro. At runtime, Bram prefers an on-disk `app/` next to the binary (for hot-reload during development) and falls back to the embedded copy. The binary deliberately does NOT use Tauri's SPA fallback asset resolver — "disastrous for XMLUI's optional code-behind probes that legitimately 404."

## Key Techniques

### Worklist Lifecycle with Hash-Verified Authorization
The centerpiece of Bram's design. A three-phase state machine (proposed → applied → committed) with cryptographic verification:

1. Agent proposes items by writing `resources/worklist.json` (metadata) + `resources/worklist-drafts/<id>.md` (before/after prose)
2. User clicks Approve/Drop/Iterate in the Worklist tab — buttons generate `{"items":[{"id":"...", "hash":"...", "feedback":"..."}]}` payload
3. `record_worklist_authorization_from_input` parses the structured turn, calls `build_worklist_authorization_record` to verify each hash against on-disk worklist items (SipHash via Rust's DefaultHasher)
4. Hash mismatch → `kind: "rejected_stale"` — agent must surface staleness and refuse to edit
5. Agent calls `/__worklist/resolve` (consume-on-read for approved records), edits files, calls `/__worklist/mutate`

**Concurrency protection**: `worklist.json` carries a `version` integer. Every write must set `version: N+1`. The PreToolUse hook denies stale writes with `reason=stale-worklist-version`.

### Dual-Provider PreToolUse Enforcement
Both Claude Code and Codex CLI get identical worklist enforcement through different mechanisms:

- **Claude**: `.claude/hooks/worklist-guard.py` (604-line Python script), registered in `.claude/settings.json`, fires on `Write|Edit`. Denies edits to project files not covered by a proposed/applied worklist item.
- **Codex**: `~/.bram/codex-worklist-guard.py`, registered in `~/.codex/config.toml` as `PreToolUse` hook with matcher `^(apply_patch|Bash|Write|Edit|mcp__.*)$`. Same coverage logic, broadened to catch Codex-specific tools.

Both hooks exempt lifecycle paths (`resources/worklist.json`, `.worklist-intent.json`, `.worklist-result.json`, etc.) and honor opt-out phrases ("just do it", "skip the worklist", "inline fix", etc.).

### Filesystem-Based Lifecycle for Sandboxed Agents
Codex's sandbox refuses loopback connections (`curl: (7)` even when Bram listens, issue #130). Rather than granting full network access, Bram uses intent/result files:

- Agent writes `resources/.worklist-intent.json` with `{nonce, route, body}`
- Host watcher drains it, dispatches through the SAME handlers as HTTP routes
- Host writes `resources/.worklist-result.json` with `{nonce, ok, status, result}`
- Both transports share consume-on-read, inflight sentinel, auth checks

This is a clever pattern: filesystem as message queue, with the host's filesystem watcher as the dispatcher.

### Turn-State Spine (docs/spine.md)
A host-owned record (`/__turn-state` payload) that arbitrates between competing signal sources:

- **PTY bytes** — fast but noisy, prone to stale redraws
- **Provider JSONL** — structured but delayed, Claude's `stop_reason: "end_turn"` / Codex's `task_complete`
- **Permission menus** — detected from PTY, supplemented by JSONL signatures
- **Inflight sentinel** — `resources/.inflight-claim.json` written by resolve/mutate handlers
- **Completion cursor** — `lastCompletionAtMs` prevents older PTY activity from relighting "working" status

The spine is "not a full event-sourcing system. It does not preserve every raw PTY byte or JSONL record. The trace does that." It exists to prevent flickering — different UI surfaces telling different stories about agent state.

### Provider-Aware Session Management
Bram reads both Claude and Codex session JSONL: Claude from `~/.claude/projects/<encoded-cwd>/`, Codex from `~/.codex/sessions/...`. The `/__sessions/latest-tail` endpoint is diff-aware: clients pass `since=<byte-offset>&sid=<session-id>`, server returns bytes `[since, EOF)` with `reset: false` for typical ~10KB deltas, or full tail with `reset: true` on session rotation.

The shared-cache pattern: `Main.xmlui`'s App-level `DataSource` consumes the envelope; `ChangeListener` branches on `reset` — `true` replaces cache, `false` appends. Cap at 1.5MB with head-trim at newline boundary. Multiple iframe components subscribe via `onLatestJsonlChange()` so each fetch fans out without re-fetching per tab.

### Voice Input via Local Whisper
Bram auto-manages a `whisper-server` child process on first record click, using `whisper.cpp`'s HTTP server at port 18080. The frontend captures audio via MediaRecorder API, sends to whisper-server as `multipart/form-data`, delivers transcript to terminal as `voice: <text>` (bracketed-paste so agents can distinguish dictated from typed input). On Windows, whisper-server runs inside WSL via `wsl.exe bash -lc`.

### Double-Buffer Iframe Swap
When hot-reloading the tools pane, Bram creates a new off-screen iframe (position absolute, 1px×1px, left: -99999px), waits for `load`, then promotes it by replacing the old iframe in the DOM. This prevents the blank-frame flash during reload. The hash route is preserved across swaps by polling `contentWindow.location.hash` every 500ms.

## Design Decisions

### Static Frontend, No Build Step
"No bundler, no `package.json`. The only build step is the Tauri/Rust build." This is a deliberate choice — the frontend is vanilla JS with vendored libraries (xterm.js, xmlui-standalone). Trade-off: no TypeScript type safety, no modern tooling. Gain: zero frontend build complexity, instant iteration, no dependency churn.

### Same-Origin iframe Architecture
Both the target app and agent tools iframe load at `tauri://localhost`, making them same-origin with the parent shell. This means `window.parent.__TAURI__.core.invoke()` works directly — no `postMessage` shim needed. The target app iframe is proxied through the Tauri scheme handler (`/__project/*` → project's HTTP server), hiding the loopback port from iframes.

Consequence: service workers don't register on macOS/Linux (custom-scheme origins aren't secure contexts in WKWebView/WebKitGTK), so Mock Service Worker and XMLUI's apiInterceptor won't work in the embedded pane on those platforms.

### XMLUI as Agent Pane Framework
The agent pane is built with XMLUI (xmlui.org), a custom reactive UI framework with its own expression engine. This is an unusual choice — most would reach for React/Vue/Svelte. Benefits: XMLUI's expression engine enforces constraints (no raw browser JS in event handlers, no async/await outside DataSource), which catches bugs at evaluation time. The XMLUI MCP server provides search_howto, component_docs, and get_prompt tools for AI-assisted development of the pane itself.

### Filesystem as State Store
No database. All coordination state lives in files under `resources/`: `worklist.json`, `.worklist-authorization.json`, `.inflight-claim.json`, `.bram-port`, `.worklist-intent.json`, `.worklist-result.json`, `worklist-drafts/`, `feedback-drafts/`, `feedback-history/`, `worklist-history/`, `bram-traces/`. The filesystem watcher (Rust `notify` crate) triggers reloads and dispatches intent files.

Trade-off: simple, transparent, git-trackable, debuggable with `cat` and `grep`. Cost: no atomic multi-key transactions, watcher latency, concurrency managed by version integers and mutexes rather than DB transactions.

### Evidence-First Debugging Philosophy
Per docs/spine.md: "Do not start by reading code and inferring behavior. Read the evidence. If the evidence proves the hypothesis, fix the path it identifies. If the evidence falsifies the hypothesis, abandon it. If the evidence is insufficient, add instrumentation before changing behavior."

The `bram-trace.log` records host-side events with structured subkind vocabulary (jsonl-fanout, heartbeat-batch, inflight-set/clear, etc.). The Inspector tap forwards XMLUI runtime events into the same trace. Together with the Status tab dashboard, this creates a falsifiable debugging surface.

### Convention File as Shared Memory
`app/__shell/conventions.md` is the canonical project convention file, loaded into every Claude Code session via `@`-import in CLAUDE.md. It's 1,089 lines covering the full worklist lifecycle, commit etiquette, voice conventions, debugging procedures, and code organization rules. The file explicitly says: "Don't save project-related memories — preferring the worklist, helper APIs, release quirks, conventions you discover, etc. Per-user memory is private to one agent on one machine; this file is shared with everyone running Bram."

This is a key insight: repo-local conventions scale better than per-user agent memory.

### Provider Symmetry with Transport Differences
Bram achieves functional parity between Claude Code and Codex CLI despite their different capabilities:

| Aspect | Claude | Codex |
|--------|--------|-------|
| Conventions | Direct @-import of conventions.md | AGENTS.md block + developer_instructions + startup seed |
| Hook | `.claude/hooks/worklist-guard.py` | `~/.bram/codex-worklist-guard.py` via config.toml |
| Lifecycle transport | Loopback curl to `127.0.0.1:<port>` | Filesystem intent/result files |
| Session JSONL | `~/.claude/projects/<encoded>/` | `~/.codex/sessions/...` |
| Turn-end detection | `stop_reason: "end_turn"` in JSONL | `task_complete` / `final_answer` in JSONL |

Same auth model, same state machine, same enforcement — different wires.

## Comparison Notes

**vs. Fleet Supervisor (sermakarevich)**: Both coordinate agents through structured work items and approval gates. Bram is a desktop shell with a real terminal and visual worklist; Fleet Supervisor is a server-side Python supervisor with web UI and Telegram HITL. Bram's PreToolUse hooks provide deterministic enforcement at the tool-call level (before the agent acts); Fleet Supervisor relies on the agent following conventions.

**vs. Broomy**: Both are desktop apps that run coding agents side-by-side. Broomy is Electron-based with a built-in IDE; Bram is Tauri-based with a real terminal and XMLUI agent pane. Bram's worklist enforcement via PreToolUse hooks is novel — most desktop agent shells rely on convention alone.

**vs. cmux**: cmux is a terminal multiplexer for managing multiple AI coding agent sessions with notification rings. Bram is a full desktop shell with structured workflow (worklist, issues, commits) and enforcement hooks. cmux focuses on session management; Bram focuses on audit trail and accountability.

**vs. Pi Coding Agent**: Pi is a CLI coding agent with a minimalist philosophy ("four tools, no MCP"). Bram is a desktop shell that wraps existing agents (Claude Code, Codex CLI) — it's a harness, not an agent. The design philosophies are different but complementary: Pi minimizes what agents can do; Bram adds structure and accountability to whatever the agent does.

**vs. "Maybe Coding Agents Don't Need a Bigger Memory" (Santi)**: Santi argues for repo-local, evidence-weighted continuity records with a resume-work-finalize lifecycle. Bram's conventions.md is exactly this pattern — repo-local shared conventions that survive session boundaries, loaded into every session via @-import. The worklist history (`resources/worklist-history/`) and feedback history (`resources/feedback-history/`) provide the audit trail.

## Tags
#tool #project #agents #coding-agent #desktop-app #tauri #rust #git #workflow #enforcement #guardrails #xmlui

## Source
Fetched 2026-06-22 from https://github.com/judell/bram (v0.2.12, Tauri desktop app, 24,439-line Rust backend)
