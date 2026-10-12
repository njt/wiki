---
url: https://tedhaley.ca/pensieve/
title: "Pensieve — Search your files, repos and agent sessions"
author: Ted Haley
date_fetched: 2026-10-12
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

Pensieve is a free, open-source (MIT) macOS tool that indexes your files, git repos (including uncommitted changes and worktrees), and coding-agent sessions locally, and exposes the index to coding agents over MCP. Its pitch: agents are only as good as the context they're given, and grep-plus-read-files wastes tool calls and tokens — one semantic search returns the few relevant passages with file and line.

The page includes a self-run benchmark: Claude Sonnet in headless Claude Code, 72 held-out code-finding tasks across four real repos (504 to 34,847 files), run twice each. With Pensieve the agent found the right file in 131/144 runs versus 123/144 with grep, used 2.0 tool calls per lookup instead of 3.4 (43% fewer), cost 18% less ($0.041 vs $0.050), and finished 21% faster. Benchmark scripts are open source. Single grep still wins on exact error messages; Pensieve wins on everything else.

Two features go beyond search: ownership maps from git blame (who knows an area, weighted by lines still present), and knowledge-gap flagging — areas written by people no longer active, with the active person who knows the area best. `pensieve integrate` wires up agents automatically and adds a "search before you grep" instruction block to CLAUDE.md-style files, since agents grep by habit. Everything runs on-device (local embedding model, index in ~/.pensieve); secrets are never indexed; AI-generated summaries are optional and can run fully locally.
