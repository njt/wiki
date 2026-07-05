---
title: "lat.md"
url: https://www.lat.md/
author: "@1st1 (github.com/1st1/lat.md)"
date_fetched: 2026-05-15
date_published: unknown
section: "Developer Tools"
---

# lat.md: Knowledge Graph for Your Codebase

Markdown-based tool creating a knowledge graph for codebases. Wiki-style markdown with source code annotations synchronize specs with implementation. Validation prevents drift.

## The Problem

AGENTS.md doesn't scale. A single flat file works for small projects, but as a codebase grows, key design decisions get buried, business logic goes undocumented, and agents hallucinate context they should be able to look up.

## The Idea

Compress domain knowledge into a graph of interconnected markdown files in a `lat.md/` directory at the project root. Sections link to each other with `[[wiki links]]`, markdown files link into the codebase (`[[src/auth.ts#validateToken]]`), source files link back with `// @lat: [[section-id]]` comments, and `lat check` ensures nothing drifts out of sync.

## Key Goals

1. **Spec that your agent keeps in sync with the codebase** — agents maintain lat files as they work
2. **Make agents understand big ideas and key business logic** — semantic search instead of endless grepping
3. **Ensure corner cases have proper high-level tests that matter** — test specs with `require-code-mention: true` enforcement
4. **Start reviewing agent diffs: focus on knowledge, not code** — review semantic changes in `lat.md/` first, code second
5. **Speed up coding by saving agents endless grepping** — agents search the knowledge graph instead of the codebase

## CLI

```bash
lat init       # scaffold lat.md/, configure agent hooks
lat check      # validate all wiki links and code refs
lat locate     # find sections by name (exact, fuzzy)
lat section    # show a section with links and refs
lat refs       # find what references a section
lat search     # semantic search via embeddings
lat expand     # expand [[refs]] in a prompt for agents
lat mcp        # start MCP server for editor integration
```

## Test Spec Enforcement

Test cases can be described as sections marked with `require-code-mention: true`. Each spec must be referenced by a `// @lat:` comment in test code. `lat check` flags any spec without a backlink, enabling review and maintenance of test coverage from the knowledge graph.

## Knowledge Retention

The context and reasoning behind prompts is usually lost after a session ends. With lat, agents capture knowledge into the graph as they work, so future sessions start with full context instead of rediscovering it from scratch.

## Configuration

Semantic search requires an OpenAI or Vercel AI Gateway API key, resolved via `LAT_LLM_KEY`, `LAT_LLM_KEY_FILE`, or `LAT_LLM_KEY_HELPER` env vars, or a config file.

## Version History

- 0.11: `lat init` supports Codex, OpenCode, and Cursor stop hook
- 0.10: `lat section` and `lat refs` show source code snippets; ripgrep-powered scanning
- 0.9: `lat init` creates a lat skill for supported agents
- 0.8: Pi coding agent integration; interactive arrow-key menus in `lat init`
- 0.7: Multi-language source links (Rust, Go, C); section structure validation
- 0.6: Source code wiki links — reference functions/classes directly: `[[src/foo.ts#myFunc]]`
- 0.5: Auto-suggest `lat init` when no `lat.md/` found

## Development

Node.js 22+, pnpm. Install globally via `npm install -g lat.md`.

## Links

- GitHub: https://github.com/1st1/lat.md
- npm: https://www.npmjs.com/package/lat.md
- Author: @1st1 on X/Twitter
