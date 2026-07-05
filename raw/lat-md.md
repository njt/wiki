---
url: https://www.lat.md/
date_fetched: 2026-07-05
backfilled: true
---

A knowledge graph for your codebase, written in markdown.

Goals

- * Spec that your agent keeps in sync with the codebase
- * Make agents understand big ideas and key business logic
- * Ensure corner cases have proper high-level tests that matter
- * Start reviewing agent diffs: focus on knowledge, not code
- * Speed up coding by saving agents endless grepping

Install

`$ npm i -g lat.md`then run lat init in your project

Key Ideas

- * Plain markdown: readable by humans, parseable by agents
- * Wiki links connect concepts into a navigable graph
- * // @lat: and # @lat: comments tie source code to specs
- * lat check ensures nothing drifts out of sync
- * lat search for semantic vector search across all sections
- Read the 

What's New

- 0.11lat init supports Codex, OpenCode, and Cursor stop hook
- 0.10lat section and lat refs show source code snippets; ripgrep-powered code scanning
- 0.9lat init creates a lat skill for supported agents
- 0.8Pi coding agent integration; interactive arrow-key menus in lat init
- 0.7Multi-language source links (Rust, Go, C); lat section and lat expand commands; section structure validation
- 0.6Source code wiki links — reference functions and classes directly from specs: [[src/foo.ts#myFunc]]
- 0.5Auto-suggest lat init when no lat.md/ found; section IDs now include h1 heading

||
