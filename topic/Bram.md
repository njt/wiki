# Bram

Bram is a Tauri desktop shell that wraps AI coding agents (Claude Code, Codex CLI) in a structured workflow with deterministic enforcement. It puts a real terminal alongside an agent tools pane with tabs for Worklist, Commits, Issues, Sessions, History, Context, Status, and Settings. The key innovation: a **hash-verified worklist lifecycle** (proposed → applied → committed) enforced by PreToolUse hooks at the tool-call level — before the agent edits a file, the hook verifies a worklist item covers that path. This makes "vibe coding" accountable without slowing it down. Built by Jon Udell, discussed in [[Jon Udell — Vibe Coding as a Team Sport]] and his earlier post "How to make best use of git and GitHub for AI-assisted software development."

## Architecture

Bram is a **Tauri 2 desktop app** with a Rust backend (24,439-line `src-tauri/src/lib.rs`) and a static vanilla JS frontend — no bundler, no package.json, no TypeScript.

**Three panes:**
- **Terminal** (left) — xterm.js connected to a real PTY (bash/powershell) running Claude Code or Codex CLI. PTY output flows as binary Uint8Array chunks over a Tauri Channel.
- **Target app** (right, hidden by default) — optional iframe proxying the project's dev server through `tauri://localhost/__project/*`. Most users view their app in their own browser.
- **Agent tools drawer** (bottom-right) — XMLUI iframe with 8 tabs driving the workflow surface.

**Five transport layers** between host (Rust) and consumers (iframes, agents):
1. Tauri scheme routing (`tauri://localhost/__*`) for browser-facing iframes
2. Internal HTTP routes (`/__worklist`, `/__commits`, `/__sessions/*`) served by `tiny_http`
3. Loopback HTTP for Claude's curl calls (port from `resources/.bram-port`)
4. Tauri IPC (`invoke()`) for direct WebView↔host commands
5. Filesystem coordination (`resources/.worklist-intent.json` / `.worklist-result.json`) for Codex, whose sandbox blocks loopback

The `app/` tree is compiled into the binary via `include_dir!` macro, with on-disk preference at runtime for hot-reload during development.

## Key Techniques

### Hash-Verified Worklist with PreToolUse Enforcement

The centerpiece. Every change request flows through a three-phase state machine:

1. **Propose** — agent writes `resources/worklist.json` (metadata) + `resources/worklist-drafts/<id>.md` (before/after prose). Each item gets a server-computed SipHash.
2. **Approve** — user clicks Approve in the Worklist tab. The button generates `{"items":[{"id":"...", "hash":"...", "feedback":"..."}]}`. The host verifies each hash against on-disk items before recording authorization in `resources/.worklist-authorization.json`.
3. **Commit** — agent edits files (PreToolUse hook checks worklist coverage), advances via `/__worklist/mutate`, user approves commit separately.

Hash mismatch → `kind: "rejected_stale"` — agent must surface staleness and refuse to edit. This prevents TOCTOU attacks where the worklist changes between approval display and agent action.

**Concurrency**: `worklist.json` carries a `version` integer. Every write must set `version: N+1`. The PreToolUse hook denies stale writes. `/__worklist/mutate` does the same bump under a serializing mutex.

### Dual-Provider Enforcement with Different Transports

Claude and Codex get identical enforcement through different mechanisms:

| | Claude Code | Codex CLI |
|---|---|---|
| Conventions | `@`-import of `conventions.md` in CLAUDE.md | AGENTS.md block + `developer_instructions` + startup seed |
| Hook | `.claude/hooks/worklist-guard.py` (604 lines) | `~/.bram/codex-worklist-guard.py` via `~/.codex/config.toml` |
| Lifecycle calls | `curl http://127.0.0.1:<port>/__worklist/resolve` | Write `resources/.worklist-intent.json`, read `.worklist-result.json` |
| Turn-end detection | `stop_reason: "end_turn"` in session JSONL | `task_complete` / `final_answer` in session JSONL |

The filesystem intent/result pattern is a clever workaround for Codex's sandbox — same handlers, different transport.

### Turn-State Spine

A host-owned record (`/__turn-state`) that arbitrates between competing signal sources to prevent UI flickering. PTY bytes are fast but noisy; provider JSONL is structured but delayed; permission menus are detected from both. The spine merges them into a single `{provider, phase, pendingMenu, lastPtyActivityAtMs, lastJsonlActivityAtMs, lastCompletionAtMs, source, reason}` payload so all UI surfaces tell the same story.

### Convention File as Shared Memory

`app/__shell/conventions.md` (1,089 lines) is loaded into every Claude Code session via `@`-import. It explicitly tells agents: "Don't save project-related memories — this file is shared with everyone running Bram." This is a key insight: repo-local conventions scale better than per-user agent memory. The file covers the full worklist lifecycle, commit etiquette, voice conventions, debugging procedures, and code organization rules.

### Provider-Aware Session Browsing with Diff-Aware Tail

The Sessions tab reads both Claude and Codex JSONL from their respective directories. The `/__sessions/latest-tail` endpoint is diff-aware: clients pass `since=<byte-offset>&sid=<session-id>`, and the server returns only new bytes (`reset: false`) for typical ~10KB deltas. Full resend only on session rotation. A shared-cache pattern (`setLatestJsonl` / `appendLatestJsonl` with 1.5MB cap) fans out to all iframe components without per-tab re-fetching.

## Design Decisions

**Static frontend, no build step**: Vanilla JS with vendored libraries (xterm.js, xmlui-standalone). Trade-off: no TypeScript type safety, no modern tooling. Gain: zero frontend build complexity, instant iteration, no dependency churn.

**Filesystem as state store**: No database. All coordination state lives in files under `resources/`. The filesystem watcher triggers reloads and dispatches intent files. Simple, git-trackable, debuggable with `cat` and `grep`. Cost: no atomic multi-key transactions, concurrency managed by version integers and mutexes.

**XMLUI for the agent pane**: An unusual choice over React/Vue/Svelte. XMLUI's expression engine enforces constraints (no raw browser JS in event handlers, no async/await outside DataSource) — catches bugs at evaluation time. The XMLUI MCP server provides AI-assistance tools for developing the pane itself.

**Evidence-first debugging**: "Do not start by reading code and inferring behavior. Read the evidence." The `bram-trace.log` with structured subkind vocabulary, Inspector tap for XMLUI runtime events, and Status tab dashboard create a falsifiable debugging surface.

**Provider symmetry, transport diversity**: Same auth model, same state machine, same enforcement across Claude and Codex — different wires. The filesystem intent/result pattern for Codex keeps the sandbox intact while achieving functional parity.

## Comparison Notes

**vs. [[Fleet Supervisor (sermakarevich)]]**: Both coordinate agents through structured work items and approval gates. Bram is a desktop shell with PreToolUse hooks providing deterministic enforcement at the tool-call level; Fleet Supervisor is a server-side Python supervisor relying on convention adherence.

**vs. [[Broomy]]**: Both are desktop apps running multiple coding agents. Broomy is Electron with a built-in IDE; Bram is Tauri with a real terminal and XMLUI agent pane. Bram's worklist enforcement via PreToolUse hooks is novel — most desktop agent shells rely on convention alone.

**vs. [[Maybe Coding Agents Don't Need a Bigger Memory]]** (Santi): Santi argues for repo-local continuity records. Bram's `conventions.md` is this pattern in practice — shared conventions that survive session boundaries, loaded into every session via `@`-import.

**vs. [[cmux]]**: cmux manages multiple agent sessions with notification rings. Bram adds structured workflow (worklist lifecycle, issue tracking, commit management) and enforcement hooks. cmux focuses on session awareness; Bram focuses on audit trail and accountability.

**vs. [[Loop Engineering]]** (Addy Osmani): Bram is loop engineering made concrete — a system that prompts agents (via conventions.md + PreToolUse hooks) instead of prompting agents yourself.

## Tags
#tool #project #agents #coding-agent #desktop-app #tauri #rust #workflow #enforcement #guardrails #git

## Source
Analyzed 2026-06-22 from https://github.com/judell/bram (v0.2.12). 24,439-line Rust backend, 1,600-line JS frontend, zero-dependency static web stack, XMLUI agent pane, dual-provider PreToolUse enforcement for Claude Code and Codex CLI.
