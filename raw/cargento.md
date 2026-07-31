---
url: https://github.com/spacedock-dev/cargento
title: Cargento — Agnostic Agent Cartography and Visualization
author: spacedock-dev
date_fetched: 2026-08-01
date_published: 2025
---

# Cargento — Agnostic Agent Cartography and Visualization

Source: https://github.com/spacedock-dev/cargento

## What it is

Cargento is a cross-harness agent cartography dashboard — a local web server that maps live coding-agent activity across nine different AI coding tools (Claude Code, Codex, Pi, Gemini CLI/Antigravity CLI, GitHub Copilot CLI, OpenCode, Cursor CLI, Goose, and Factory Droid). It reads each tool's local session stores read-only, assembles a self-refreshing HTML dashboard, and fires desktop notifications when a Claude session is blocked waiting for human input.

The project is a single Python plugin (`cargento`) distributed across four harness marketplaces (Claude Code, Codex, Antigravity, Gemini CLI). The runtime is stdlib-only Python 3.11+ — no dependencies to install alongside it.

## Architecture

The plugin has a markdown-first structure: the skill body lives in `cargento/skills/cargento/SKILL.md` and the dashboard runtime is an importable Python package (`cargento_runtime/`) beside it. Four harness manifests (`plugin.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `gemini-extension.json`) wrap the same skill body for each platform.

The launcher (`server.py`) is seven lines that import from `cargento_runtime.cli.main`. The runtime package has been extracted from what was originally a single 7,357-line file into ~25 modules with explicit dependency direction:

### Core runtime modules

- **`config.py`** — Frozen `RuntimeConfig` dataclass built once at the process boundary. Resolves store roots per platform (XDG, APPDATA, home-relative), carries every tunable limit and threshold. Nothing downstream reads the environment or `sys.platform`.
- **`state.py`** — Mutable `RuntimeState` dataclass holding bounded caches (metadata, titles, CWDs, turn scanners, Spacedock entities), locks (hook, cache, scanner, collection memo), popup state, and the server start stamp.
- **`aggregate.py`** — `HarnessSpec` registry (discover + collect per harness), the per-harness failure boundary (`try/except Exception` so one broken store can't take the dashboard down), and the `Application` class that binds config, state, and injected services (notifiers, diagnostic sink, clock).
- **`http_api.py`** — `CargentoHTTPServer` (a `ThreadingHTTPServer` subclass) with port hijack protection on Windows (`SO_EXCLUSIVEADDRUSE`), loopback-only enforcement via Host/Origin/Sec-Fetch headers (defeats DNS rebinding and cross-site fetch), and three routes: `/` (assembled page), `/api/data` (JSON collection output), `/api/health` (liveness with pid/port/started).
- **`lifecycle.py`** — State file management (`~/.cargento/cargento-<port>.json`), port probing (discriminates Cargento from foreign processes), stop (POST `/api/shutdown`, waits for port release), double-fork daemon detach on POSIX, and re-spawn on Windows (no fork).
- **`cli.py`** — argparse surface, runtime assembly, and three distinct serve branches: Windows daemon re-spawn (parent never constructs a server), POSIX fork (binds before detaching), and foreground.
- **`sessions.py`** — Session identity (`base_session` template, `Session` type alias), freshness logic with future-skew rejection, display ID widening (grows only within harness+project groups that actually collide), project label extraction (two-path segments shared across all harnesses), and duration formatting.
- **`records.py`** — Defensive parsing of untrusted records from disk: safe text sanitization, timestamp normalization (epoch milliseconds vs seconds, ISO 8601), turn signal detection (per-harness rules for what counts as a prompt/end), Gemini snapshot deduplication, and record fingerprinting via BLAKE2b.
- **`turns.py`** — Incremental whole-file turn scanner shared across JSONL harnesses: reads only bytes appended since last call, carries state in `turn_scan` cache, handles long turns via backward then forward scanning, tracks per-harness turn boundaries, and computes naive median-based ETA estimates.
- **`transcripts.py`** — First-line metadata cache (immutable line-1 JSON parsed once per file), prompt title extraction (slash-command unwrapping, path collapse, word-boundary clipping), and per-harness tail analyzers (Codex, Gemini, Copilot, Droid).
- **`claude_data.py`** — Claude-specific transcript analysis shared by the collector and the hook path: subagent classification (top-level `agentName` records vs legacy `subagents/` directory), Spacedock agent setting detection, and session CWD extraction.
- **`spacedock.py`** — Project-side workflow cartography: reads workflow README frontmatter for stage lists, entity state files for current stage, boot envelopes for directory paths. Every read goes through `open_regular()` (OS-level symlink refusal + inode identity check), is bounded in bytes, and is cached per `(path, mtime_ns, size)`.
- **`io.py`** — Bounded filesystem I/O: tail reads, first-line JSON, reverse-line scanning (chunked backward read for turn boundary location), safe globbing across store roots, read-only SQLite URI construction, and the diagnostic sink.
- **`notifications.py`** — Hook payload handling (POST `/api/notify`), popup policy (per-session and global cooldown floors), native macOS notifier via `osascript display notification`, and hook generation tracking.
- **`collectors/`** — Nine independent collector modules, one per harness. Each exports `discover(config, state) -> bool` and `collect(config, state, now, window_hours, show_all) -> list[Session]`. Claude is the only collector that takes an additional `popup_notifier` argument (bound at registry assembly).
- **`diagnostics.py`** — `--diagnose` reporting: walks every store root, reports presence and readability, distinguishes "no store" from "store present but unreadable" from "store read, no recent sessions."
- **`web/page.py`** — Loads `index.html`, `styles.css`, `app.js` from the package directory, assembles them into a single byte payload. The contract validator proves the assets resolve from an installed plugin with no repository.
- **`web/app.js`** — Client-side JavaScript: 5-second polling, two display modes (card stack / calm ledger), keyboard navigation, notification permission handling, rate sparklines, and the stop button arming sequence.

### Dependency rules

Dependencies run inward (lower layers never import higher ones), enforced by an AST-based import graph test with an explicit allowlist. Collectors may not import each other or `aggregate`. `TYPE_CHECKING` imports count — a dependency that exists only for annotations is still a dependency.

### Configuration/state/service separation

Three objects with deliberately different lifetimes:
- `RuntimeConfig` — frozen, built once at process boundary
- `RuntimeState` — mutable, holds caches, scanner offsets, locks, popup state
- `Application` — binds one config and one state to injected services (notifiers, sink, clock)

This means two servers can run in one interpreter without sharing caches or notification state — a contract test verifies this.

## Key techniques

### Per-harness failure boundary
Each harness collector runs inside its own `try/except Exception` in `Application.collect()`. A single malformed transcript file or a corrupt SQLite database can only take down its own harness, not the entire dashboard. The `--diagnose` command separately distinguishes "store absent," "store present but unreadable," and "store read but no recent sessions" — the collectors can't because they skip unreadable stores silently by design.

### Future-timestamp rejection
`age()` in `sessions.py` rejects timestamps implausibly ahead of the clock (beyond `future_skew_tolerance_sec`, default 120s). This prevents a session restored from backup with a clock-skewed mtime from reading as perpetually "Working." The rejection threshold is distinct from the freshness window — a future timestamp is not clamped to zero (which would read "just now") but treated as implausible. `newest_plausible()` filters future values before taking the max, so one skewed record doesn't hide good evidence.

### Incremental turn scanning
`scan_turns()` in `turns.py` tracks turn boundaries by reading only bytes appended since the last call, carrying state in a per-path scanner cache. For transcripts where the current turn is longer than the tail window (the prompt is buried beyond the tail bytes), it locates the turn boundary by reading backward in chunks (`reverse_lines()`), then processes the bounded tail forward. The scanner is serialized under a lock so concurrent `/api/data` requests don't double-advance the position and double-count durations.

### Display ID widening
`assign_display_ids()` grows each session's display ID only within its `(harness, project)` group where a collision actually exists. Codex hands out UUIDv7 whose leading 48 bits are a millisecond timestamp — a fan-out launched in one directory shares its leading hex. Widening per harness would drag every unrelated row out to the width one colliding fan-out needed. The algorithm iterates: widen one character at a time until all prefixes in the group are distinct, bounded by the longest full ID.

### Gemini snapshot deduplication
Gemini CLI stores resumed sessions as `$set.messages` snapshots that repeat earlier messages. `incremental_gemini_records()` tracks the last seen message fingerprint and the snapshot count, so each refresh only processes new messages. A bounded set of message fingerprints (`gemini_seen_entries`, default 2048) prevents unbounded growth.

### Spacedock workflow cartography from passive reads
The Spacedock module reconstructs workflow state without any API: it reads `spacedock status --boot` output from Claude transcript records (only in `tool_result` blocks, never conversation text), extracts absolute directory paths, reads workflow README frontmatter for stage lists, and reads entity state files for current stage. The boot envelope's `dispatchable` is treated as a stale snapshot — the entity state directory is the authoritative source. Every project read goes through `open_regular()` which refuses symlinks and non-regular files, validates inode identity between stat and open (to catch parent-directory swaps), and is bounded in bytes.

### Collection memo with lock-held-through-collection
`collect_json()` holds the collection memo lock through the entire scan, not just the cache check. On a `ThreadingHTTPServer`, concurrent requests share one filesystem/SQLite scan rather than stampeding cold cache entries. The memo TTL is 2.5 seconds — shorter than the 5-second refresh, so two refreshes in a row never return the same stale data.

### Turn gap reset
The turn scanner detects quiet stretches longer than `turn_gap_reset_sec` (default 300s) inside a turn — permission prompts, open questions, sleep — and re-anchors the elapsed clock at the post-gap event. This means "elapsed" reflects generation time, not waiting time. A mid-turn quiet stretch doesn't count against the ETA.

### Loopback enforcement
The HTTP handler enforces loopback-only access through three checks:
1. `Host` header must resolve to `127.0.0.1`, `localhost`, or `::1` (defeats DNS rebinding)
2. `Sec-Fetch-Site: cross-site` is rejected unless it's a top-level document navigation (defeats cross-site fetch from web pages, but allows clicking a link)
3. `Origin` header is compared against the listening port, not just the hostname (defeats same-site attacks from other local servers)

The `normalize_host()` function handles bracketed IPv6, bare IPv6, and port-bearing Host headers without naive `rsplit(":", 1)` — which would turn `[::1]` into `[:`.

## Design decisions

### Optimized for: correctness of passive signal reconstruction
The hardest problem Cargento solves is reconstructing agent state from stores that were never designed to be read by a third party. The project invests heavily in defensive parsing (every field is untyped JSON from disk), timestamp normalization (sub-second, millisecond, and microsecond epochs), and future-skew rejection. The design documents explicitly call out failure modes like "a session restored from backup reads as perpetually Working" and design against them.

### Sacrificed: live token data for four harnesses
Copilot, OpenCode, Cursor, and Droid sessions always contribute 0 to the token rate tile because their stores don't expose usable live token totals. The dashboard shows a dash rather than synthesizing a number.

### Sacrificed: needs-input detection for non-Claude harnesses
Only Claude Code sessions get needs-input detection (via `AskUserQuestion` in transcripts or Notification hook POSTs). The SKILL.md explicitly states "Claude only — other harnesses have no needs-input detection."

### Sacrificed: Linux/Windows native notifications
The native notifier works on macOS (via `osascript display notification`). On Linux and Windows, the browser must be open for notifications — the page raises browser notifications instead. A hook path still delivers no popup when no tab is open on these platforms.

### Optimized for: testability through dependency injection
The entire runtime takes its environment as arguments: `platform_name`, `os_name`, `environ`, `home`, and the clock are all injected. This means one test runner exercises Linux, macOS, and Windows branches without platform-specific CI. Configuration is frozen, state is mutable, services are injected — the architecture document explicitly frames this as "the payoff is testability of the awkward cases."

### Optimized for: adding harnesses cheaply
Adding a harness is two things: a module under `collectors/` exporting `discover` and `collect`, and a row in `default_harnesses()`. CONTRIBUTING.md owns the walkthrough. The collector contract is one signature for all nine harnesses.

## Comparison notes

### vs AgentsView
[[AgentsView]] is another local-first analytics dashboard for 24+ coding agents. Cargento is narrower (9 harnesses) but deeper — it does session-level state reconstruction (working/idle/needs-input), per-harness turn boundary detection, per-session ETA estimation, subagent tracking with named pills, Spacedock workflow stage strips, and native desktop notifications. Cargento is also a plugin distributed across four harness marketplaces, while AgentsView appears to be a standalone tool.

### vs Omnigent
[[Introducing Omnigent]] wraps existing agents in a uniform API for composition and security. Cargento is the inverse: it reads agent stores passively and provides visibility rather than control. Both are multi-harness, but Omnigent is an orchestration layer while Cargento is an observability layer.

### vs Subspace
[[Subspace]] is another Claude Code plugin from the same organization (spacedock-dev) — it opens Markdown files in a native TUI for human review. Cargento is the observability companion: Subspace handles the human-in-the-loop review step, Cargento shows you what all your agents are doing.

### vs Steering Claude Code
[[Steering Claude Code]] catalogs Claude Code's instruction-delivery mechanisms. Cargento is the runtime counterpart — it shows you what Claude Code (and eight other harnesses) are actually doing, not how to configure them.

### vs Hook-based dashboards
Most agent monitoring tools require an SDK or an explicit integration. Cargento works with zero configuration for any harness whose store is discoverable — it reads files that already exist. The Notification hook path (POST to `/api/notify`) is opt-in and only needed for needs-input detection without a browser tab open.

## Security model
- Server binds 127.0.0.1 only — never 0.0.0.0
- Loopback enforcement via Host, Origin, and Sec-Fetch-Site headers
- Windows: `SO_EXCLUSIVEADDRUSE` prevents port hijacking by other local processes
- Project reads (Spacedock) go through `open_regular()`: symlink refusal, inode identity check, byte bounds
- All harness stores are read-only — transcripts, SQLite (mode=ro), task files
- No data leaves the machine
- Defensive parsing everywhere: untyped JSON from disk, Unicode replacement, field type coercion, RecursionError caught on JSON parse

## Codebase stats
- ~141K total lines across Python (runtime + tests + scripts)
- ~25 runtime modules under `cargento_runtime/`
- 9 collector modules, one per harness
- 19 configurable limits and thresholds in `RuntimeConfig`
- Stdlib-only: no pip dependencies for the runtime
- Python 3.11+ floor
- Tests: unit tests with coverage threshold enforced in CI, contract tests for module identity, import graph, and asset loading

---
Source: https://github.com/spacedock-dev/cargento
Analyzed: 2026-08-01
