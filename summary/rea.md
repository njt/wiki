---
url: https://github.com/morluto/rea
title: "REA: Reverse Engineer Anything"
author: morluto
date_fetched: 2026-10-10
date_published: 
topics:
  - mcp-and-tool-protocols
  - coding-agents-and-frameworks
---

REA (`rea-agents` on npm, MIT, ~40,000 stars) is a local MCP server plus CLI that gives any coding agent — Claude Code, Codex, Cursor, Gemini CLI, and others — the tooling to reverse engineer software: native binaries via Hopper/Ghidra/IDA, JavaScript and Electron apps, .NET assemblies, EVM bytecode, Android APKs, firmware, HAR captures, and browser or process runtime behaviour. `npx rea-agents setup` registers the MCP server with the chosen agent and installs a bundled "reverse-engineer-anything" skill.

The architecture is a provider-adapter design: a TypeScript core (~137K lines of non-test TS, 284 test files) exposes a tool catalog via MCP, while per-engine Python/Java/Swift "bridge" scripts run inside Hopper, Ghidra, pwntools, JADX, and mitmproxy and talk back over an authenticated socket protocol. Everything runs locally; results carry evidence and stated limitations rather than bare conclusions.

Showcases emphasise verification over extraction: DX-Ball's audio-pan routine reconstructed in C passes 3,205 test cases and reproduces all 63 bytes of the original compiled x86 function; Notion's clipboard bridge is traced from renderer through preload IPC to the main process; TH04's DOS bullet-ring maths is checked against period-compiler output.
