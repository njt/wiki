---
url: https://github.com/colbymchenry/codegraph
title: "CodeGraph — Semantic Code Intelligence for Coding Agents"
author: Colby McHenry (colbymchenry)
date_fetched: 2026-11-06
date_published: 2026 (v1.6.2 at fetch)
topics:
  - coding-agents-and-frameworks
  - mcp-and-tool-protocols
---

CodeGraph is a local code-intelligence layer that sits between an AI coding agent and a codebase. `codegraph init` parses a project into a SQLite knowledge graph (`.codegraph/codegraph.db`) of symbols, edges, imports, and framework routes, then keeps it fresh with a debounced native file watcher. Agents reach it through an MCP server exposing — deliberately — a single tool, `codegraph_explore`, which returns the verbatim source of relevant symbols plus call paths (including dynamic-dispatch hops) and a blast-radius summary in one call.

The headline claims are benchmark-backed with unusual honesty: across seven open-source repos (VS Code, Django, Tokio, OkHttp, …) with/without the graph on Claude Opus 4.8, the median agent made 88% fewer tool calls, read zero files, and was 53% faster and 44% cheaper with the graph. The README also publishes the counter-metric: CodeGraph leaves ~80% more retrieval context resident at session end (one dense payload that stays in the window vs many small evicted reads), and admits earlier benchmark figures were contaminated until they blocked the CLI in the control arm too — 26 of 28 control runs had found it on PATH.

Implementation is where the interest lies: a Rust napi-rs kernel does tree-sitter parse+extract for 20 languages with one boundary crossing per file (graphs verified byte-for-byte identical to the portable TypeScript engine, with per-file WASM fallback); ~90k lines of TypeScript handle resolution, framework synthesizers (17 router frameworks, Swift↔ObjC, React Native/TurboModules/Expo bridges, Tauri), and a shared per-project daemon so N MCP clients amortize one watcher, one SQLite writer, and one warm-up. 100% local, no API keys, MIT.
