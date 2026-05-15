# sem

Semantic version control CLI. Instead of line-level diffs, sem analyzes code at the entity level -- functions, classes, methods -- using tree-sitter. Entity-level diff, blame, graph, and impact analysis across 26 programming languages. When an agent or human asks "what changed?", the answer is "the `processOrder` function was modified and `validateInput` was renamed," not "lines 47-92 changed."

---

## Key Themes

#tool #version-control #semantic-analysis #tree-sitter #agent-tooling

The core commands tell the story:
- `sem diff`: Entity-level change detection with rename recognition
- `sem impact`: What breaks if this entity changes?
- `sem blame`: Who's responsible for this function?
- `sem log`: How has this entity evolved over time?
- `sem context`: LLM-optimized summaries with token budgeting

The three-phase matching (exact ID -> structural hash -> fuzzy similarity at 80% threshold) is the key technical innovation. It distinguishes cosmetic changes (whitespace, comment updates) from logic modifications through structural hashing. This means a reformatted function doesn't show up as "changed" unless the logic actually changed.

The AI-first design is explicit: JSON output, MCP server with six tools, token-budgeted context generation. This is built for agents to consume, not just humans.

## Critical Analysis

This solves a real problem for agent-driven development. When an agent needs to understand what changed in a PR, line diffs are noisy and wasteful of context window space. Entity-level diffs give the agent precisely the information it needs: which functions were added, modified, renamed, or deleted, and what depends on them.

The `sem impact` command is underrated -- cross-file dependency analysis answering "what breaks if I change this?" is exactly what agents need before making modifications. Most agents today just change things and hope the tests catch problems. Impact analysis lets them reason about consequences before acting.

26 languages with tree-sitter is broad coverage. The fallback to chunk-based diffing for unsupported languages is pragmatic.

2k stars and 93.3% Rust for a version control tool suggests a performance-conscious community. Dual MIT/Apache-2.0 licensing removes any adoption friction.

See also [[graphify]] and [[lat.md]] for other approaches to making codebase structure navigable.

---
*Sources: [[raw/sem]]*
*Last updated: 2026-05-14*
