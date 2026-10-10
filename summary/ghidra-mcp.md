---
url: https://github.com/bethington/ghidra-mcp
title: "Ghidra MCP Server"
author: bethington
date_fetched: 2026-10-10
date_published: ongoing (v7.0.0 in development; v5.4.x documented)
topics:
  - coding-agents-and-frameworks
  - mcp-and-tool-protocols
---

bethington's Ghidra MCP Server is a two-part system for letting AI agents drive Ghidra, the NSA's open-source reverse engineering suite: a Java Ghidra plugin (~44K lines) that runs an HTTP/Unix-socket server inside Ghidra and exposes 209 annotated tool endpoints, plus a Python MCP bridge (~5K lines) that translates the Model Context Protocol onto that HTTP surface for clients like Claude Code and Cursor. The README claims 3x the tool count of competing Ghidra-MCP implementations, with full write access — renaming, retyping, commenting, structure creation, script execution, P-code emulation, and live debugger integration — not just reads.

The distinctive claim is v5.0's "opinionated by design" posture: naming conventions, type-safety checks, and documentation standards are moved out of prompts and into the tool layer, where they can be enforced. Writes run through a three-tier policy — auto-fix (silently applies Hungarian-prefix conventions, e.g. `count` on a `uint32` becomes `dwCount`), warn (change goes through with a naming warning), or reject (no-op type changes are blocked). An `analyze_function_completeness` tool scores documentation coverage 0–100%, forgiving unfixable compiler artifacts and log-scaling category penalties.

Other notable machinery: cross-binary documentation transfer via normalized SHA-256 opcode hashes (addresses replaced with placeholders so identical functions match across binary versions); batch operations claiming 93% fewer API calls; P-code emulation for brute-force API hash resolution in milliseconds; and a lazy tool-loading default in the bridge, because advertising all 209 tools in one `tools/list` breaks Gemini's constrained-decoding compiler outright. Security is localhost-first: script-execution endpoints default off after v5.4.1, non-loopback binds require a bearer token, and filesystem endpoints get a path-traversal root.

---

*Sources: [[raw/ghidra-mcp]]*
