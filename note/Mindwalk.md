# Mindwalk

A 3D visualization tool that replays coding-agent sessions as light moving through a night map of your codebase. One Go binary reads Claude Code and Codex session logs — where the agent searched, read, and edited, the map glows; everything else stays dark. The tool makes an agent's understanding of a task visible at a glance, without sending any data off your machine.

---

## Architecture

Mindwalk is a single Go binary (~9K lines) with an embedded React/Three.js frontend. It runs a localhost HTTP server that scans session directories, parses JSONL logs, and serves normalized traces and citymaps to the browser.

The system produces three independent first-class artifacts, each with its own JSON Schema (`schema/`):

1. **Trace** — a normalized event stream from raw agent session logs. Two adapter backends (`internal/adapter/claudecode/` and `internal/adapter/codex/`) implement the `adapter.Source` interface, converting heterogeneous formats into a common schema. Every tool call is classified into one of six actions (search, read, edit, exec, verify, other) via `actionFor()` in `internal/adapter/adapter.go`, a ~30-tool classifier that parses structured tool inputs and shell command strings.

2. **Citymap** — a deterministic squarified-treemap layout of the repository (`internal/citymap/`). The weight function is `sqrt(max(lines, bytes/4096, 16))`, and the same tree always produces the same map, making replays comparable across sessions. Ghost files (touched by the session but no longer in the repo) are included as wireframe outlines.

3. **Report** — an LLM judge's evidence-anchored findings about a session (`internal/judge/`). The judge receives a synthesized evidence document (task wording, precomputed stats, per-event narrative) — never the raw session log. Findings cite specific event sequence numbers; dimension verdicts are rolled up mechanically from finding severities, not decided by the LLM.

The server (`internal/server/`) joins these artifacts with multi-layer fingerprint-based caching and inflight request deduplication (concurrent requests for the same session share one parse). All cache staleness checks use file size + modTime fingerprints.

## Key Techniques

### Tool Classification by Input, Not Output

The adapter classifies tool *calls* (what the agent asked to do), not tool *results*. `Read` → read action, `Write`/`Edit` → edit action. Shell commands are the exception: `actionFor()` parses shell pipelines to distinguish `grep` (search) from `cat` (read) from `npm test` (verify), with conservative guards — `find -exec` is exec, not search.

### Weak Target Hedging

Paths extracted from command strings (rather than structured tool inputs like `file_path`) are marked `weak=true`. Weak targets are filtered if the file doesn't exist on disk, and their presence downgrades the observability grade from `exact` to `estimated`. This is a clever hedging mechanism: "we think this shell command referenced this file, but we're not sure."

### Deterministic Layout for Cross-Session Comparability

The squarified treemap is deterministic — same tree, same layout every time. This means two sessions on the same repo produce visually comparable maps, and the "terrain" view where height encodes attention depth is meaningful because the underlying layout is stable.

### Sealed Judge Subprocess

The LLM judge runs in a stripped environment to prevent prompt injection from the evaluated session reaching tools. For the Codex judge, this means 14 explicit `--disable` flags plus `--sandbox read-only`. For Claude Code: `--tools "" --strict-mcp-config --setting-sources "" --no-session-persistence`. The judge's findings are validated: every evidence sequence number is checked against the actual trace, and findings with no valid evidence are dropped.

### Regression Rate as Wandering Signal

The stats compute `regressionRate = repeatedReads / readEvents`, where a "repeated read" is reading a file without an intervening edit. This is a lightweight proxy for "the agent is re-reading the same content" — a signal of context loss or wandering. The rate carries an observability grade that tracks how reliable the underlying read detection is.

### Agent Graph with Link Quality Metadata

Subagent parent-child relationships carry a `linkQuality`/`linkMethod` pair — `exact` (matched via tool_use_id), `derived` (matched by directory proximity), or `unavailable` (launch observed, trace missing). This is unusually honest metadata: downstream consumers decide how much weight to give each relationship.

## Design Decisions

**Single binary, zero runtime deps (beyond git).** The web frontend is embedded via `//go:embed`. Install is `curl | sh`. No npm, Python, Docker, or database.

**Viewing is free, judging costs.** The hard architectural boundary: browsing sessions sends nothing anywhere. LLM evaluation is opt-in, explicit, and runs through the user's own CLI with their own API key.

**No database, just files.** Session lists are built by directory scans. Traces are cached in memory. Reports are cached as flat JSON files. This simplifies deployment but means the server re-scans on startup (mitigated by parallel scanning across CPU cores).

**Deterministic stats, LLM findings only.** Stats are precomputed; the judge is instructed to trust them. The judge outputs only findings; verdicts are mechanical rollups. This prevents LLM contradictions with the numbers while still enabling qualitative observations.

**Adapter pattern over canonical format.** Rather than defining a canonical session format, mindwalk adapts to whatever Claude Code and Codex actually produce. New agent formats can be added without touching rendering, citymap, or judge systems.

**Ghost files as storytelling.** Files the session touched that no longer exist render as wireframe outlines — visually communicating "the agent worked with stale information." The attention terrain (height by touch depth × revisits, smoothed per-frame) similarly makes agent focus patterns legible at a glance.

## Comparison Notes

Mindwalk occupies a unique niche: it's a visual replay tool with a spatial metaphor (the codebase as a city) rather than a timeline or dashboard.

- [[claude-replay]] produces self-contained HTML replays of single sessions; mindwalk adds the 3D citymap metaphor, cross-session comparability, agent graphs, and LLM evaluation
- [[session-analysis]] computes wall time, tokens, and cost; mindwalk goes deeper into attention patterns (fovea/parafovea, regression rate, churn)
- [[atifact]] converts session logs to the ATIF interchange format; mindwalk is a full visualization suite, not a converter
- [[AgentsView]] provides analytics dashboards across many agents; mindwalk focuses on detailed per-session visual replay on a codebase map
- [[engineering-notebook]] auto-generates engineering diaries from sessions; mindwalk provides the visual attention analysis that complements narrative summaries

The squarified treemap layout, attention-height terrain, ghost file visualization, and sealed LLM judge are features with no direct parallel in other session analysis tools.

---
*Sources: [[raw/mindwalk]]*
*Last updated: 2026-07-18*
