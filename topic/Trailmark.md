# Trailmark

A source code analysis tool from Trail of Bits that parses code into a queryable graph database of functions, classes, calls, and semantic annotations. Built for security analysis and vulnerability identification, but useful for anyone who needs to understand large codebases structurally.

---

## Key Themes

#graph-db #security #devtools

Trailmark's three-phase pipeline (parse, index, query) turns source code into a graph you can interrogate. The query operations are where it gets interesting: `callers_of()` and `callees_of()` for direct relationships, `paths_between()` for call chain analysis, and `entrypoint_paths_to()` for attack surface mapping. `complexity_hotspots()` flags functions that are likely to harbor bugs based on cyclomatic complexity.

The language-agnostic approach (25+ languages via tree-sitter) means it works on polyglot codebases -- the common case in real organizations. The semantic annotation system lets security researchers mark assumptions and findings directly on the graph, which is a clever way to build institutional knowledge about a codebase.

Integration with SARIF (the static analysis results format) means you can overlay findings from other tools onto the same graph. This is the kind of composability that makes a tool actually useful in practice rather than just impressive in a demo.

Connects to [[NornicDB]] (graph database for code-like structures) and [[sql-crack]] (different approach to code visualization -- SQL-specific rather than general-purpose).

## Critical Analysis

Trail of Bits has credibility in security tooling, and the graph-based approach to code analysis is sound. The Python >= 3.12 requirement and rustworkx dependency suggest this is built for performance. What's missing from the README is a clear story about scale -- how does it handle million-line codebases? The 378 stars suggest early adoption. The attack surface analysis use case is the killer feature; most code analysis tools focus on finding bugs, but mapping the attack surface is a higher-leverage activity for security teams.

---
*Sources: [[summary/trailmark]]*
*Last updated: 2026-05-14*
