---
title: "sem"
url: https://github.com/ataraxy-labs/sem
date_fetched: 2026-05-14
section: "Random"
---

# sem: Semantic Version Control CLI

Git-based tool analyzing code at entity level rather than line-by-line. Uses tree-sitter to identify functions, classes, methods, enabling semantic diffs, blame tracking, and impact analysis across 26 languages.

Core value: Instead of "lines x-y changed," identifies specific entities modified, renamed, or deleted.

Commands:
- sem diff: Entity-level change detection with rename recognition and structural hashing
- sem impact: Cross-file dependency analysis (what breaks if entity changes)
- sem blame: Entity-level responsibility tracking
- sem log: Historical evolution of specific entities
- sem entities: List all code structures
- sem context: LLM-optimized summaries with token budgeting

Three-phase matching: Exact ID -> structural hash comparison -> fuzzy similarity (>80% token overlap).

Distinguishes cosmetic changes from logic modifications via structural hashing. No setup required -- works in any Git repo.

AI-first design: JSON output, MCP server with six tools, token-budgeted context for LLM consumption.

Languages: TypeScript, Python, Go, Rust, Java, C++, plus JSON, YAML, Markdown. Unsupported languages fall back to chunk-based diffing.

Install via Homebrew, npm, cargo, Docker, or source. 2k stars, 93.3% Rust, dual MIT/Apache-2.0 licensed.
