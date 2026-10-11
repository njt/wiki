# Agent Standup Internals

A deep-dive into the code behind [[Agent Standup]]: how a zero-dependency Node server turns Claude Code's JSONL session files into a live multi-agent "team call", and what its engineering choices reveal about reading agent state without paying tokens for it.

---

Agent Standup reads the transcripts Claude Code already writes under `~/.claude/projects` (honouring `CLAUDE_CONFIG_DIR`), merges the main session file with one file per sub-agent, and reconstructs the delegation tree as a replayable timeline of events. The joke is the standup call; the substance is that the entire observability layer is classical file parsing, with model calls only for opt-in extras (French translation, deadpan "quips"). It ships the same server four ways — VS Code extension, CLI, Electron desktop app, `npx` — because the surface should follow the user, not the other way around.

## Architecture

- **`server.js` (607 lines, zero npm dependencies)** — the whole backend: raw `http`, file reading, timeline building, translation workers, TTS, routing. One file, deliberately. The frontend is one 211 KB `public/index.html`, no framework, no build step; the repo's build scripts exist only for the landing page, CLI bundling, and i18n extraction.
- **`extension.js`** — VS Code host: starts the server in-process on a free port per window, shows it in a webview (strict CSP, iframe with a tiny postMessage relay), and polls `boardRows()` directly to drive a status-bar indicator with counts of working / needs-you / your-turn sessions.
- **`bin/agent-standup.js`** — the CLI: `serve`, `status [--json]`, and `statusline`, which consumes Claude Code's status-line JSON on stdin and prints `✋ 2 needs you · ⚙ 3 working`-style output.
- **`desktop/main.js`** — Electron shell around the same server, with a tray/menu-bar indicator and notifications when the "needs you" count rises. Sandboxed, context-isolated; navigation locked to localhost.
- **`kokoro.js`** — opt-in neural voices (Kokoro-82M via ONNX Runtime), installed on demand (~430 MB packages, ~155 MB model) with a one-line-at-a-time worker process and bounded restarts.
- **`demo.js`** — a fictional five-agent session fixing a slow login page, same event shape as real timelines, so the demo needs no real data and no translation service.

## Key techniques

- **Event model as the core abstraction.** Every parsed transcript line becomes one of ~9 event kinds: `user`, `say`, `tool(s)`, `order`, `report`, `finish`, `usage`, `results`, `compact`. Delegation is reconstructed structurally: an `Agent`/`Task` tool-use id in the parent's file is matched against each sub-agent's `.meta.json` (`toolUseId`) to build `parent[to]`, which lets every sub-agent's last `say` be retyped as a `report` addressed to its owner. The agent tree is derived, not recorded.
- **Incremental transcript reader** (`readNew`): each JSONL file has a persistent state `{offset, tail, toolName, msgUsage}`; only newly appended bytes are read, in ≤16 MB chunks, and a half-written final line is carried over until complete. One malformed record is skipped without killing the timeline (`try { parseRecord } catch {}`).
- **Cheap board over many sessions** (`parseTail`): instead of full parses, read the last 192 KB of each transcript, track pending tool-use ids, and classify state heuristically — `Task` pending = `waiting`; `AskUserQuestion` after 4 s or `Edit/Write` pending after 15 s = `blocked` (the agent is waiting on *you*); assistant turn with no pending tools = `your turn`. Titles come from cached tail reads: custom title beats AI title beats first user prompt, with regexes over raw bytes rather than full JSON parsing.
- **Tail-scanning for media**: tool screenshots are extracted by scanning the transcript file in 8 MB windows for the tool id, with a 12-entry memo — nothing outside the session files is ever read or served.
- **Warm translation workers**: `claude -p` cold-starts in ~6 s, so two persistent `claude` workers run `--input-format stream-json`, are recycled after 30 messages (context growth), idle-killed after 10 minutes, and load-balanced by queue depth; a fast path hits the API directly when `ANTHROPIC_API_KEY` exists. All translations are cached per string.
- **Security posture rare in local tools**: localhost host/Origin checks against DNS rebinding, `Sec-Fetch-Site` filtering on side-effect endpoints, 256 KB body caps, strict path regexes (`/^[\w.\-]+$/`, no `..`), and `lstat` so symlinks in the characters folder are never served.
- **Single-audio-owner lease**: multiple browser windows poll `/api/audio`; the last claimant wins and must keep heartbeating, so only one window ever plays sound.

## Design decisions

- **Zero dependencies** is the load-bearing choice: it makes `npx @double_geste/agent-standup` a 30-second try-before-install path and keeps the tool auditable — a privacy-sensitive reader of your prompts should be readable in one sitting. The cost is hand-rolled HTTP, MIME, and spawn plumbing.
- **Observation over inference.** The board and timeline cost zero tokens; anything needing a model (translation, jokes) is explicit and opt-in, using your own login rather than shipping a key. This is the same economics argument as the [[Agent Standup]] landing page, but proven in code: even the "blocked" heuristic exists to avoid asking a model what an agent is doing.
- **Derived vs recorded state.** Nothing is stored; every restart re-derives the timeline from the transcripts, which stay the single source of truth. The trade-off is a 3-session LRU timeline cache with file-stamp invalidation and a "rebuild at most every max(250 ms, 2× build time)" throttle for huge sessions.
- **Anthropomorphisation as UX**: silence thresholds become 💤, long commands pulse amber, sub-agents dissolve 2 minutes after reporting. The display logic encodes opinions about what "stuck" means — and those thresholds are where the product lives.

## Comparison notes

- The website note [[Agent Standup]] judged the product from its feature list; this repo shows the judgment survives contact with the code — the "zero token cost" claim is literally true, and the tree/replay data model is exactly as implied.
- [[AgentsView]] does broad multi-agent analytics over the same kind of session files (24+ formats, SQLite, search); Agent Standup goes deep on one format and one live metaphor instead — a real split in local agent observability between coverage and legibility.
- [[claude-replay]] produces static shareable HTML replays; Agent Standup's replay is live-first (follow the newest session, stickiness rules to avoid session-stealing mid-line) with export as a secondary. Both prove the same insight: JSONL transcripts are the raw material nobody should read raw.
- Compared with harness-engineering write-ups like [[The Harness Is the Company]], this is the rare artifact that treats the harness's *own* state (sub-agent files, compact boundaries, task notifications, `system-reminder` wrappers) as a public interface to be parsed — its `HARNESS` regex stripping harness wrappers is a de facto spec of Claude Code's transcript format.

Tags: #tool #project #agents #claude-code #observability #local-first

---
*Sources: [[raw/agent-standup-github]], [[summary/agent-standup-github]]*
*Last updated: 2026-10-11*
