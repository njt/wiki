---
url: https://github.com/Atoll92/agent-standup
title: "Agent Standup (GitHub repository)"
author: Atoll92 (DoubleGeste)
date_fetched: 2026-10-11
date_published: 2026 (v1.24.1)
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

The repo behind [[Agent Standup]]: a zero-dependency Node.js tool that turns Claude Code's local session files into a live "team call" visualization of the agent tree. One 607-line `server.js` (plain `http`, no npm dependencies) reads the JSONL transcripts Claude Code already writes under `~/.claude/projects` (honouring `CLAUDE_CONFIG_DIR`), parses the main session file plus one file per sub-agent, and merges them into a single timestamped event timeline. The same server ships four ways: a VS Code extension (`extension.js`), a CLI (`bin/agent-standup.js`, including a `statusline` mode that reads Claude Code's status-line JSON on stdin), an Electron desktop app with a tray indicator, and `npx` for one-shot use.

The core abstraction is the **event timeline**: each transcript line becomes one of a handful of event kinds (`user`, `say`, `tool`, `order`, `report`, `finish`, `usage`, `results`), where `order`/`report` events reconstruct who delegated to whom by matching the `Agent`/`Task` tool-use id to the sub-agent whose `.meta.json` records it. Consecutive tool calls by one agent merge into a single activity "beat". The reader is incremental — each file is parsed once and only appended bytes are read, in ≤16 MB chunks with careful handling of half-written last lines, and a 3-session LRU cache with per-file stamps.

Deliberate economics: observation costs zero tokens. The only model calls are opt-in (French translation and "quips"), made via the local `claude` CLI with warm persistent workers, or the API directly if `ANTHROPIC_API_KEY` is set. Board state for many parallel sessions is derived from a 192 KB tail read per file, classifying sessions into `working / waiting / blocked / turn / idle` heuristically from the last pending tool call. Security is unusually careful for a local server: localhost-only host/origin checks, `Sec-Fetch-Site` filtering on side-effect endpoints, path validation, and `lstat` to refuse symlinked character files. The whole frontend is one 211 KB `public/index.html` — no framework, no build step.
