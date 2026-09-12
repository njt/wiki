# Understand-Anything

A Claude Code plugin that builds a structured knowledge graph from any codebase by combining deterministic tree-sitter parsing with LLM-based semantic analysis. The output is a JSON graph (21 node types, 35 edge types) rendered in an interactive React dashboard. Uniquely, it persists as a living artifact in `.understand-anything/` and stays current via git hook-triggered incremental updates. Unlike one-shot codebase explainers, this embeds the analysis into the development workflow.

## Architecture

The plugin runs as a multi-agent pipeline orchestrated by Claude Code, with a TypeScript core library providing shared types, validation, and persistence.

**Agent pipeline** (8 specialized agents):
1. **project-scanner**: Discovers all files via `git ls-files`, detects languages/frameworks, resolves imports across 12 languages (TypeScript to PHP PSR-4), estimates complexity. Uses a deterministic Node.js script for file discovery.
2. **file-analyzer**: Two-phase batched analysis — tree-sitter extracts functions/classes/imports deterministically, then LLM adds summaries, tags, and complexity ratings. Each file becomes a `file:` node; significant functions and classes become `function:` and `class:` child nodes.
3. **architecture-analyzer**: Computes import adjacency matrices, fan-in/fan-out, intra-group density, and cross-category dependencies via script, then the LLM assigns files to 3-10 logical layers.
4. **domain-analyzer**: Extracts business domains → flows → steps (three-level hierarchy) from either preprocessed context or existing graph.
5. **assemble-reviewer**: Post-merge quality review — recovers dropped nodes/edges, fills cross-batch gaps, sanity-checks mechanical fixes.
6. **tour-builder**: Generates guided codebase tours ordered by topological dependency sort (Kahn's algorithm).
7. **article-analyzer**: For knowledge-kind graphs — extracts entities, claims, and implicit relationships from markdown/wiki articles.
8. **graph-reviewer / knowledge-graph-guide**: Final review and educational guide generation.

**Core library** (`packages/core/src/`):
- `schema.ts`: Four-tier validation pipeline — sanitize (nulls→defaults), normalize (LLM aliases→canonical), auto-fix (missing fields, type coercion, weight clamping), validate (Zod schemas + referential integrity)
- `fingerprint.ts`: SHA-256 content hashing + structural signature comparison to classify changes as NONE / COSMETIC / STRUCTURAL
- `change-classifier.ts`: Decision matrix mapping change analysis to update actions — SKIP, PARTIAL_UPDATE, ARCHITECTURE_UPDATE, FULL_UPDATE, with concrete thresholds (>30 files or >50% = full rebuild)
- `plugins/`: 10 tree-sitter language extractors (TS/JS, Python, Go, Rust, Java, Ruby, PHP, C/C++, C#) + 13 non-code parsers (Markdown, YAML, JSON, TOML, .env, Dockerfile, SQL, GraphQL, Protobuf, Terraform, Makefile, Shell)

**Hooks**: PostToolUse triggers incremental updates on git commits. SessionStart checks staleness against stored commit hash.

**Dashboard**: React + ReactFlow with ELK layout, Louvain community detection, multi-language support, persona selector, and mobile-responsive design.

## Key techniques

**Three-tier LLM output sanitization**: The validation pipeline is unusually defense-in-depth. It doesn't trust LLM output at all — every value is sanitized, aliases are mapped (80+ edge type aliases, 75+ node type aliases), missing fields get defaults, and even string weights get coerced to numbers. This is pragmatic engineering: prompt engineering alone can't guarantee consistent output, so the system accepts the mess and cleans it up deterministically.

**Two-phase agent design (script + LLM judgment)**: Every agent follows a script-first pattern. Deterministic computation (file discovery, tree-sitter parsing, adjacency matrices, density calculations) runs in Node.js. The LLM handles only semantic interpretation (summaries, tags, layer naming). This constrains the LLM to tasks it's good at while keeping structural data trustworthy.

**Fingerprint incremental updates**: Rather than re-analyzing everything, the system compares SHA-256 content hashes plus structural signatures (function/class/import/export names and signatures). If content changed but structure is identical, it's classified as COSMETIC (no graph update needed). Only STRUCTURAL changes (new/removed functions, changed imports) trigger re-analysis of affected files. The change classifier then determines update scope with explicit thresholds.

**Edge weight as ordering**: In domain graphs, `flow_step` edges use monotonically increasing fractional weights (0.1, 0.2, 0.3...) to encode step ordering within a flow, avoiding the need for a separate ordering field.

**Cross-language import resolution**: One of the most comprehensive import resolvers in open-source — handles 12 languages with language-specific patterns: Python absolute imports with `__init__.py` probing, Go module path matching, PHP PSR-4 namespace resolution, Ruby load-path searching, TypeScript path alias resolution via tsconfig paths.

**Dual layer detection**: Heuristic directory-pattern matching (50+ patterns across ecosystems) runs deterministically first. The LLM can then refine. The heuristic covers edge cases like Django's `blueprints`, Python's `serializers`, and Go's `cmd/` entry points.

## Design decisions

**Optimized for deterministic structure, LLM semantics**: Tree-sitter owns structure; LLM owns interpretation. The right split — neither component alone would work.

**Sacrificed speed for thoroughness**: Full pipeline runs multiple agents sequentially with per-file LLM calls. Expensive but produces higher-quality output than single-pass approaches.

**LLM output treated as adversarial**: The alias tables and validation pipeline reveal hard-won experience. Rather than begging the LLM for canonical values, the system accepts common variants and maps them deterministically.

**JSON artifact over direct LLM context**: Unlike tools that inject codebase context into prompts, this produces a standalone JSON graph enabling the dashboard, incremental updates, and reuse. The trade-off is that the graph is a lossy representation.

**Weakness — no deterministic call graph**: `calls` edges are LLM-inferred ("infer from imports + function names when confident") despite tree-sitter being available for call graph extraction. Import edges are deterministic; call edges are not.

**Weakness — dashboard is read-only**: No human correction loop back into the graph. Corrections require JSON editing or re-analysis.

## Comparison notes

- **vs [[DeepWiki]]**: DeepWiki is on-demand exploration with line citations. Understand-Anything is a persistent plugin that lives with the repo and updates incrementally. DeepWiki for one-shot; UA for ongoing orientation.
- **vs [[Trailmark]]**: Trailmark queries source code as a security graph. UA is broader (configs, docs, infra) but less security-focused.
- **vs [[GraphRAG]]**: GraphRAG builds entity-relationship graphs from text for RAG. UA builds file-function-class containment and import/call graphs from code for developer orientation.
- **vs [[lat.md]]**: lat.md produces markdown knowledge graphs that are git-diff friendly. UA produces richer JSON (35 edge types, 21 node types) with an interactive dashboard, at the cost of diff readability.
- **vs [[graphify]]**: graphify handles multimodal sources (code, PDFs, screenshots). UA is code-only but much deeper in structural analysis.

## Tags

#tool #agents #claude-code #knowledge-graph #code-analysis #visualization #typescript

## Source

GitHub: [Lum1104/Understand-Anything](https://github.com/Lum1104/Understand-Anything)
Fetched: 2026-05-18
