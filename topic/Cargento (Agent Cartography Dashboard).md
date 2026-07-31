# Cargento (Agent Cartography Dashboard)

A cross-harness agent observability dashboard that maps live coding-agent activity across nine AI coding tools — Claude Code, Codex, Pi, Gemini CLI, Copilot CLI, OpenCode, Cursor CLI, Goose, and Factory Droid — into a single local web UI. Reads each tool's session stores read-only, reconstructs agent state (working/idle/needs-input) without any SDK or integration, and fires desktop notifications when a session is blocked waiting for human input. Stdlib-only Python, distributed as one plugin across four harness marketplaces.

---

## Architecture

Cargento is a markdown-first plugin distributed to four harnesses (Claude Code, Codex, Antigravity, Gemini CLI) from one repository. The shared skill body lives in `cargento/skills/cargento/SKILL.md`; the dashboard runtime is an importable Python package (`cargento_runtime/`) beside it. Four harness manifests wrap the same skill body for each platform.

The launcher is seven lines. The runtime was extracted from a single 7,357-line file into ~25 modules with enforced dependency direction (lower layers never import higher ones, verified by an AST-based import graph test).

### Three-object separation

- **`RuntimeConfig`** (`config.py:30-86`) — Frozen dataclass built once at process boundary. Resolves store roots per platform, carries every tunable limit (19 thresholds from `tail_bytes=400_000` to `display_id_len=8`). Nothing downstream reads the environment.
- **`RuntimeState`** (`state.py:20-48`) — Mutable dataclass: bounded caches (metadata, titles, turn scanners, Spacedock entities), locks, popup state, collection memo.
- **`Application`** (`aggregate.py:83-166`) — Binds one config and one state to injected services (native notifier, popup notifier, diagnostic sink, clock). Two servers can run in one interpreter without sharing caches.

### Collector registry (`aggregate.py:37-80`)

Nine `HarnessSpec` rows, each naming a collector module's `discover` and `collect`. The contract is one signature for all: `(config, state, now, window_hours, show_all)`. Claude is the only collector that notifies during collection — a transcript-detected transition into needs-input has no HTTP request behind it — so its row gets the popup notifier bound at registry assembly.

Every collector runs inside its own `try/except Exception` in `Application.collect()` — one broken store can only take down its own harness, never the whole dashboard.

### HTTP server (`http_api.py`)

`CargentoHTTPServer` (a `ThreadingHTTPServer` subclass) with three routes: `/` (assembled page), `/api/data` (JSON collection), `/api/health` (liveness). Loopback enforcement via three checks: `Host` header resolution, `Sec-Fetch-Site` cross-site rejection (except top-level document navigations), and `Origin` port comparison. On Windows, `SO_EXCLUSIVEADDRUSE` prevents port hijacking by other local processes.

### Daemon lifecycle (`lifecycle.py` and `cli.py`)

Three distinct serve branches: Windows daemon parent re-spawns a foreground child and awaits its pid (never constructs a server — the child owns the bind); POSIX daemon binds before forking (so a busy port explains itself on the terminal that asked); foreground binds and serves in the same process. State is tracked in `~/.cargento/cargento-<port>.json`, written atomically through a temp file and `os.replace`.

### Frontend (`web/page.py`, `web/app.js`)

The page is assembled server-side: `index.html` has `{{CARGENTO_STYLES}}` and `{{CARGENTO_APP}}` slots that get replaced with the contents of `styles.css` and `app.js`. Two display modes render the same `/api/data` payload: a card stack and a calm ledger. Client-side: 5-second polling, keyboard navigation, rate sparklines, notification permission handling.

## Key techniques

### Future-timestamp rejection (`sessions.py:106-127`)
Timestamps implausibly ahead of the clock (beyond 120s skew tolerance) are rejected outright, not clamped to zero. A session restored from backup with a clock-skewed mtime would otherwise read as perpetually "Working." `newest_plausible()` filters future values before taking the max, so one skewed record doesn't hide good evidence.

### Incremental turn scanning (`turns.py`)
Turn boundaries are tracked by reading only bytes appended since the last call, sharing state via a per-path scanner cache serialized under a lock. For turns longer than the tail window, the scanner locates the boundary by reading backward in chunks (`reverse_lines()`), then processes the bounded tail forward. Turn gaps >5 minutes (permission prompts, sleep) re-anchor the elapsed clock so "elapsed" reflects generation time, not waiting.

### Display ID widening (`sessions.py:204-236`)
Session IDs grow only within the `(harness, project)` group where a collision exists. Codex UUIDv7 prefixes collide on fan-outs; widening per harness would drag every unrelated row out to the width one colliding fan-out needed. The algorithm iterates one character at a time until all prefixes in the group are distinct.

### Gemini snapshot deduplication (`records.py:89-108`)
Gemini CLI stores resumed sessions as `$set.messages` snapshots that repeat earlier messages. `incremental_gemini_records()` tracks the last-seen message fingerprint and snapshot count, so each refresh only processes new messages.

### Spacedock workflow cartography from passive reads (`spacedock.py`)
Reconstructs workflow state without any API: reads `spacedock status --boot` output from Claude transcripts (only in `tool_result` blocks, never conversation text), extracts absolute directory paths, reads workflow README frontmatter for stage lists, and reads entity state files for current stage. Every project read goes through `open_regular()` — OS-level symlink refusal + inode identity check between stat and open — and is bounded in bytes. The boot envelope's `dispatchable` is treated as a stale snapshot; the entity state directory is authoritative.

### Collection memo with lock-held-through-collection (`aggregate.py:168-180`)
The collection memo lock is held through the entire scan, not just the cache check. On a `ThreadingHTTPServer`, concurrent requests share one filesystem/SQLite scan rather than stampeding cold cache entries. The memo TTL (2.5s) is shorter than the refresh interval (5s).

### Defensive parsing everywhere (`records.py`)
Every field from disk is untyped JSON: `safe_text()` sanitizes with Unicode replacement and control-character stripping, `parse_ts()` handles sub-second/millisecond/microsecond epochs, `extract_text()` does recursive depth-limited extraction from nested dicts/lists, and `record_fingerprint()` uses BLAKE2b for content-addressed deduplication.

### Turn gap reset (`turns.py:31-35`)
Quiet stretches >5 minutes inside a turn re-anchor the elapsed clock at the post-gap event. This means a session parked on a permission prompt for 20 minutes doesn't report its turn as "elapsed: 20m, ETA: ?" — it correctly distinguishes generation time from waiting time.

## Design decisions

**Optimized for correctness of passive signal reconstruction.** The hardest problem is reconstructing agent state from stores never designed for third-party reading. The code invests heavily in timestamp normalization, future-skew rejection, defensive field coercion, and per-harness turn signal detection rules.

**Sacrificed: live token data for four harnesses.** Copilot, OpenCode, Cursor, and Droid sessions always contribute 0 to the token rate because their stores don't expose usable live token totals.

**Sacrificed: needs-input detection for non-Claude harnesses.** Only Claude Code sessions get needs-input detection. The SKILL.md states it plainly: "Claude only — other harnesses have no needs-input detection."

**Sacrificed: Linux/Windows native notifications.** The native notifier works on macOS via `osascript`. On other platforms, the browser must be open for notifications.

**Optimized for testability.** The entire runtime takes its environment as arguments — `platform_name`, `os_name`, `environ`, `home`, clock — so one test runner exercises all platform branches. Configuration is frozen, state is mutable, services are injected.

**Optimized for adding harnesses cheaply.** Adding a harness is two things: a module under `collectors/` and a row in `default_harnesses()`. The collector contract is one signature for all nine.

## Comparison notes

**vs [[AgentsView]]:** Both are local-first agent activity dashboards. AgentsView claims 24+ agents; Cargento covers 9 but with deeper per-session state reconstruction — turn boundaries, ETA estimation, subagent tracking with named pills, Spacedock workflow stage strips, and native desktop notifications.

**vs [[Introducing Omnigent]]:** Omnigent wraps agents in a uniform API for composition and security — an orchestration layer. Cargento is the inverse: passive observability. Both are multi-harness, but they answer different questions (control vs visibility).

**vs [[Subspace]]:** Both are spacedock-dev projects distributed as Claude Code/Codex plugins. Subspace handles the human-in-the-loop review step; Cargento handles cross-harness observability. They're complementary.

**vs [[Components of a Coding Agent]]:** That page argues "the harness matters more than the model." Cargento is a harness-level observability tool — it tells you what the harness is doing across nine different implementations, making the harness itself visible.

**vs [[Loop Engineering]]:** Addy Osmani's taxonomy includes "designing systems that prompt agents instead of prompting agents yourself." Cargento is one of those systems — it passively watches agents work so the human can operate at the decision layer.

## Security model

- Binds 127.0.0.1 only — never 0.0.0.0
- Loopback enforcement via Host, Origin, and Sec-Fetch-Site headers
- Windows: `SO_EXCLUSIVEADDRUSE` prevents port hijacking
- Project reads (Spacedock) through `open_regular()`: symlink refusal, inode identity check, byte bounds
- All harness stores read-only: transcripts, SQLite (mode=ro), task files
- No data leaves the machine
- Defensive parsing: untyped JSON, Unicode replacement, field coercion, RecursionError caught on parse

---

*Sources: [[raw/cargento]]*
*Last updated: 2026-08-01*
