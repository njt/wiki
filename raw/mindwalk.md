---
url: https://github.com/cosmtrek/mindwalk
title: mindwalk
author: Ricko Yu (cosmtrek)
date_fetched: 2026-07-18
date_published: 2026
---

# mindwalk

A visualization tool that replays coding-agent sessions on a 3D map of your codebase. Draws the repository as a night map and plays the session back as light moving through it — where the agent searched, read, and edited, the map glows; everything else stays dark. One Go binary reads Claude Code and Codex session logs, fully local; viewing sends nothing anywhere.

Source: https://github.com/cosmtrek/mindwalk

## Architecture

### Language & Stack
- Go 1.25 backend (single binary, ~11K lines production code, ~2K lines tests)
- React + Three.js frontend (TypeScript, Vite, embedded via `//go:embed`)
- Squarified treemap layout for the citymap
- SQLite-like fingerprint caching (file size + modTime) for session traces and summaries
- SHA-256 content-based keys for session identity and agent graph nodes

### Artifact Pipeline

The system produces three independent JSON artifacts, each with its own JSON Schema in `schema/`:

1. **Trace** — session log normalized into an ordered stream of file-touch events with action classification (search/read/edit/exec/verify/other), touch hierarchy per file (hit < read < edit), and timeline marks (compaction, user-message, subagent). Two adapter backends: `internal/adapter/claudecode/` and `internal/adapter/codex/`. Stats are computed deterministically via `model.ComputeStats()`.

2. **Citymap** — deterministic file/directory layout using Bruls/Huizing/van Wijk's squarified treemap algorithm. Weight function: `sqrt(max(lines, bytes/4096, 16))`. Layout is deterministic: the same tree always produces the same map, making replays comparable across sessions. Ghost files (files touched by the session that no longer exist in the repo) are included as wireframe remnants.

3. **Report** — LLM judge evaluation of a session trajectory. The judge sees a synthesized evidence document (BuildInput), NOT the raw session log. The judge outputs only findings; dimension verdicts are rolled up mechanically from finding severities. Reports are cached in `~/.mindwalk/reports` with SHA-256 input digest for freshness.

### Adapter Pattern

The core abstraction is `adapter.Source` interface (Harness, SessionDir, ListSessions, Summarize, Parse):

- **Claude Code adapter** (`internal/adapter/claudecode/`): Reads the JSONL session format with its type/timestamp/message structure. Error detection is `ObservabilityExact` because the harness provides a structural `is_error` flag. Subagent discovery via `subagents/` directory sidecar `.meta.json` files.

- **Codex adapter** (`internal/adapter/codex/`): Handles the more complex Codex session format with `session_meta`, `turn_context`, `response_item`, and `event_msg` line types. Error detection is `ObservabilityEstimated` because failures must be inferred from output text and exit codes. Handles apply_patch result enrichment from `patch_apply_end` event_msg payloads.

Both adapters share the shared `adapter.BuildEvent()` function which uses `actionFor()` and `targetsFor()` to classify every tool call into an action category and extract file paths from tool inputs and outputs.

### Tool Classifier (`internal/adapter/adapter.go`)

The `actionFor()` function is a ~30-tool classifier mapping agent tools to six actions:

- **read**: Read
- **edit**: Write, Edit, MultiEdit, NotebookEdit, apply_patch
- **search**: Grep, Glob, LS, view_image
- **exec**: Bash, exec_command, js, js_repl — classified further based on command content
- **verify**: Bash/exec_command containing test commands (go test, npm test, etc.)
- **other**: everything else

For Bash/exec commands, the classifier parses shell pipelines to distinguish search commands (grep, rg, find, fd, ls) from read commands (cat, head, tail, sed -n) from verify commands (test runners). The `sedReadsOnly()` function specifically checks for the `sed -n '…p' file` read idiom vs the `-i` in-place edit flag.

### Agent Graph System

Both adapters implement `AgentGraphSource` to build a tree of subagent relationships:

- **Claude Code**: Discovers subagent sessions via `<session-id>/subagents/` directory, matches `.meta.json` sidecar files to session log files, correlates launches via `Agent`/`Task` tool_use IDs. Link quality tiers: `exact` (matched via tool_use_id), `derived` (matched via subagents directory), `unavailable` (launch observed but no trace found).

- **Codex**: Walks `session_meta` payloads to discover `parent_thread_id` relationships. Links via `agent_id` in `spawn_agent` output (exact) or `task_name` (legacy/derived). Handles ambiguous root IDs where non-main sessions share the root's session ID.

Agent node ordering uses `adapter.OrderAgentNodesPreorder()` which guarantees every parent's complete subtree is contiguous — sorting siblings by main-first, then launch sequence, then label, then ID.

### Judge System (`internal/judge/`)

The judge runs a local agent CLI sealed against prompt injection:

- **No tools, no MCP servers, no user/project settings, no session persistence**
- Claude Code judge: `-p --tools "" --strict-mcp-config --setting-sources "" --output-format json --no-session-persistence`
- Codex judge: `--ephemeral --ignore-user-config --ignore-rules --sandbox read-only` with 14 specific tool disables
- Runs in `~/.mindwalk/judge/` — a neutral workdir with no repository or project instructions
- Retries once on invalid output (unparseable JSON, unknown severities)
- Cached reports use `InputDigest` (SHA-256 of full evidence document) for freshness — not just event count

### Evidence Document (`BuildInput`)

The judge receives a synthesized document containing:
- Session metadata (harness, model, cwd, timestamps)
- User messages (first + last 11, each truncated to 600 runes) to capture the task wording
- Precomputed deterministic stats (JSON block — "trust these numbers")
- Per-event narrative pipeline (seq | action | targets | summary), capped at 2000 events
- Compaction marks interleaved

### Frontend (`web/`)

React 18 + Three.js 3D rendering:
- `CityScene.tsx`: Instanced mesh terrain with attention-height columns. Heights grow/shrink smoothly per frame. Touch states mapped to colors (unvisited/ghost/seen/read/edited/selected). FNV-1a hash jitter on unvisited tiles for visual texture. Static map mode colors by LOC tier (grey→orange→purple→red).
- `TreeScene.tsx`: Radial tree layout as alternative visualization
- `playback/reducer.ts`: State machine managing playback position, touch states, visit counts
- `Hud.tsx`: Friction signals (error rate, churn, edits after last verify)
- `AgentsPanel.tsx`: Agent lens picker for subagent replay

### Server (`internal/server/server.go`)

Single localhost HTTP server with:
- Session scanning with parallel file summarization across `runtime.NumCPU()` workers
- Multi-layer caching: session list (5s TTL), trace/citymap (10min TTL, LRU eviction at 16 entries), repo maps (30s TTL)
- Inflight deduplication: concurrent requests for the same session share one parse
- Fingerprint-based staleness: file size + modTime comparisons
- Report state badges (none/running/done/stale/failed) computed per session

### Observability Grading

Every derived stat carries an observability grade:
- `exact`: harness records the signal structurally (Claude Code's is_error flag)
- `estimated`: inferred from command/output text (Codex error detection)
- `unavailable`: log carries no usable signal

This grading feeds the verdict rollup: if `reads` observability is `unavailable`, the `exploration` and `wandering` dimensions automatically score `insufficient-data`.

## Key Techniques

### Tools Is Better Than Content Classification

Rather than trying to parse tool output to understand what happened, mindwalk classifies tools by their *input* — what the agent asked to do. `Read` → read action, `Write`/`Edit` → edit action. Shell commands (`Bash`, `exec_command`) are the exception: the classifier parses the command string to distinguish `grep` (search) from `cat` (read) from `npm test` (verify). This is a deliberate trade: input classification is cheap and reliable but misses semantic intent; shell parsing catches the important cases without false positives.

### Weak Target Tracking

Paths extracted from command strings (rather than structured tool inputs) are marked `weak=true`. Weak targets are filtered out when the file doesn't actually exist on disk (via `os.Stat`), and their presence degrades the observability grade from `exact` to `estimated`. This is a clever hedging mechanism: "we think this shell command referenced this file, but we're not sure."

### Deterministic Layout for Comparability

The squarified treemap is deterministic — same tree → same layout every time. This is deliberate engineering: it means two sessions on the same repo produce visually comparable maps, and the "terrain" view where height encodes attention depth is meaningful because the underlying layout is stable.

### Fingerprint Caching Everywhere

The server uses file fingerprinting (size + modTime) at every cache level: session summaries, parsed traces, citymaps, repo maps, agent graphs. The code is meticulous about fingerprint matching — a cached trace is only reused if the source file hasn't changed. Even inflight loads track which fingerprint they started with, so a request that arrives mid-parse knows whether to reuse or retry.

### Ghost Files

Files that the session touched but that no longer exist in the repo are included as `ghost: true` entries in the citymap. They render as wireframe outlines, visually showing "the agent touched something that's gone now." This is a small detail that reveals a lot about session quality — was the agent working with stale information?

### Sealed Judge Subprocess

The judge CLI runs in an environment stripped of every capability an injected prompt could reach. For the Codex judge, this means 14 explicit `--disable` flags plus `--sandbox read-only` plus `--ephemeral` plus `--ignore-user-config` and `--ignore-rules`. The explicit tool stripping is defense-in-depth beyond the sandbox. The Claude Code version uses `--tools "" --strict-mcp-config --setting-sources ""` to the same effect.

### Re-read Regression Rate

The stats compute `regressionRate = repeatedReads / readEvents`, where a "repeated read" is reading a file without an intervening edit. This is a lightweight proxy for "the agent is re-reading the same content" — a signal of wandering or context loss. The rate is only reliable when read observability is `exact` (no weak reads).

### Churn Detection

`churnFiles` counts files edited in 3+ events, and `editsAfterLastVerify` counts edits since the last verify event. Together these give a quick read on session discipline: high churn + unverified edits = the agent kept iterating without checking its work.

### Agent Graph Link Quality

Each subagent node carries a `linkQuality` and `linkMethod` pair. This is unusually honest metadata: it tells you *how* the adapter arrived at this parent-child relationship. `exact` (matched by tool_use_id in output) is trustworthy; `derived` (matched by directory proximity) is a best guess; `unavailable` means we saw the launch but never found the result. Downstream consumers can decide how much weight to give each relationship.

## Design Decisions

### Single Binary, No External Dependencies

The entire tool ships as one Go binary with the web frontend embedded via `//go:embed`. The only runtime dependency beyond Go is `git` (for `git ls-files` and `git rev-parse`). No npm, no Python, no Docker. This is a deliberate choice for zero-friction distribution — the install script is `curl | sh`.

### Viewing Is Free, Judging Costs

The architectural boundary between viewing and judging is absolute. Viewing sessions never sends anything anywhere — the Go server runs on localhost, adapters parse local files, the frontend renders client-side. Judgment (the LLM evaluation) is opt-in, explicit, and runs through the user's own CLI with their own API key. The evaluation is the one network call in the system, and the README is explicit about what data leaves the machine.

### Adapter Pattern Over Canonical Format

Rather than defining a canonical session format and requiring agents to output it, mindwalk adapts to whatever format Claude Code and Codex actually produce. The adapter interface is clean (5 methods), and new agent formats can be added without touching the rendering, citymap, or judge systems. This is the right call for a tool that's downstream of platforms that change their log format.

### Deterministic Stats, LLM Findings Only

The judge never computes verdicts — verdicts are mechanical rollups of finding severities. Stats are precomputed and the judge is instructed to trust them. This separation prevents the LLM from having contradictory opinions about what the numbers say, while still letting it provide qualitative observations.

### No Database, Just Files

Session lists are built by scanning directories. Traces are cached in memory. Reports are cached as JSON files in `~/.mindwalk/reports/`. There's no database, no embedded key-value store, no on-disk index. This simplifies deployment enormously but means the server must re-scan on startup.

### Ghost Storytelling

The ghost file feature reveals the tool's attitude: it's not just a debugging tool, it's a *storytelling* tool. Ghost files tell a visual story about sessions that operated on stale data. The attention terrain (height by touch depth × revisits, smoothed per-frame) is similarly narrative — mountains grow where the agent lingered.

## Comparison Notes

- **vs. claude-replay**: claude-replay produces self-contained HTML replays of a single session; mindwalk adds the 3D citymap metaphor, cross-session comparability, agent graphs, and LLM evaluation.
- **vs. session-analysis**: session-analysis computes wall time, tokens, and cost from session logs; mindwalk goes deeper into attention patterns (fovea/parafovea, regression rate, churn) and adds visual replay.
- **vs. atifact**: atifact converts session logs to the ATIF trajectory format for interchange; mindwalk is a full visualization and analysis suite, not a format converter.
- **vs. AgentsView**: AgentsView provides analytics dashboards across many agents; mindwalk focuses on detailed per-session visual replay on a codebase map.

## Code Metrics

- ~9,200 lines of production Go across `cmd/`, `internal/adapter/`, `internal/citymap/`, `internal/judge/`, `internal/model/`, `internal/server/`, `internal/textutil/`
- ~2,200 lines of Go tests
- ~700 lines of TypeScript (React + Three.js frontend)
- ~170 lines of JSON Schema definitions (trace, citymap, report, agent-graph)
- Single external dependency: `github.com/santhosh-tekuri/jsonschema/v6` (for schema validation)
- MIT License
