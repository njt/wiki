---
url: https://github.com/Lum1104/Understand-Anything
title: Understand-Anything
author: Lum1104
date_fetched: 2026-05-18
date_published: 2025-10-12
topics:
  - misc
---

## Full architectural analysis

### Overview

Understand-Anything is a Claude Code plugin (v2.7.3) that builds a structured knowledge graph from any codebase and renders it in an interactive dashboard. It combines deterministic tree-sitter parsing with LLM-based semantic analysis through a multi-agent pipeline. The output persists in `.understand-anything/knowledge-graph.json` and stays current via git hook-triggered incremental updates.

### Architecture

**Plugin structure**: A monorepo with two packages:
- `@understand-anything/core` (v0.1.0): The TypeScript library containing all shared types, schema validation, graph building, search, fingerprinting, change classification, language/framework registries, and plugin system
- `@understand-anything/skill` (v2.7.3): The Claude Code plugin layer — agent definitions, hooks, and the dashboard

**Agent pipeline** (8 specialized agents):
1. `project-scanner` — Discovers all project files via git ls-files, detects languages/frameworks, resolves imports across 12 languages, estimates complexity. Uses a Node.js script for deterministic file discovery + LLM for description.
2. `file-analyzer` — Batched source file analysis. Phase 1: tree-sitter structural extraction (bundled script). Phase 2: LLM creates knowledge graph nodes/edges with summaries, tags, complexity ratings.
3. `architecture-analyzer` — Identifies logical layers by computing import adjacency matrices, fan-in/fan-out, intra-group density, and cross-category dependencies. Phase 1: structural computation script. Phase 2: LLM semantic layer assignment.
4. `domain-analyzer` — Extracts business domains, flows, and steps (three-level hierarchy). Uses either preprocessed domain context or existing knowledge graph as input.
5. `article-analyzer` — Extracts entities, claims, and implicit relationships from markdown/wiki articles (for knowledge-kind graphs).
6. `assemble-reviewer` — Post-merge quality review: recovers dropped nodes/edges, fills cross-batch gaps, sanity-checks script fixes.
7. `tour-builder` — Generates guided codebase tours ordered by dependency flow.
8. `graph-reviewer` / `knowledge-graph-guide` — Final review and educational guide generation.

**Core library modules** (packages/core/src/):
- `types.ts` — 21 node types, 35 edge types across 8 categories, KnowledgeGraph schema
- `schema.ts` — Zod validation with 4-tier pipeline: sanitize → normalize aliases → auto-fix → validate with referential integrity checking. Auto-fix handles missing fields, type coercion, alias normalization, weight clamping
- `analyzer/graph-builder.ts` — Builder pattern for constructing KnowledgeGraph objects with deduplication
- `analyzer/llm-analyzer.ts` — Prompt builders and response parsers for file-level and project-level LLM analysis
- `analyzer/layer-detector.ts` — Dual heuristic (directory pattern matching) + LLM (semantic layer detection)
- `analyzer/tour-generator.ts` — LLM prompt builder + heuristic fallback using Kahn's topological sort
- `analyzer/normalize-graph.ts` — LLM output normalization for node IDs, complexity values, batch output
- `analyzer/language-lesson.ts` — Language-specific educational content generation
- `search.ts` — Fuse.js fuzzy search with weighted fields and extended search (OR semantics)
- `embedding-search.ts` — Cosine similarity search against stored embeddings
- `fingerprint.ts` — SHA-256 content hashing + structural signature comparison (functions, classes, imports, exports) to classify changes as NONE/COSMETIC/STRUCTURAL
- `change-classifier.ts` — Decision matrix mapping change analysis to update actions: SKIP / PARTIAL_UPDATE / ARCHITECTURE_UPDATE / FULL_UPDATE
- `staleness.ts` — Git diff-based staleness detection and incremental graph merge
- `persistence/index.ts` — File I/O for knowledge-graph.json, domain-graph.json, meta.json, fingerprints.json, config.json with absolute path sanitization
- `plugins/registry.ts` — Plugin registry mapping languages to tree-sitter extractors
- `plugins/tree-sitter-plugin.ts` — Unified tree-sitter integration across 10 languages
- `plugins/extractors/` — 10 language extractors (TypeScript, Python, Go, Rust, Java, Ruby, PHP, C/C++, C#) + base extractor
- `plugins/parsers/` — 13 non-code parsers (Markdown, YAML, JSON, TOML, .env, Dockerfile, SQL, GraphQL, Protobuf, Terraform, Makefile, Shell)
- `languages/configs/` — 30+ language configs with tree-sitter grammar bindings
- `languages/frameworks/` — 11 framework configs (React, Vue, Express, Gin, Rails, Spring, Flask, FastAPI, Django, Next.js)
- `ignore-filter.ts` / `ignore-generator.ts` — .gitignore-compatible file exclusion with .understandignore support

**Dashboard** (packages/dashboard/):
- React + Vite + ReactFlow for interactive graph visualization
- ELK layout engine for automatic graph arrangement
- Louvain community detection for clustering
- Edge aggregation for reducing visual noise
- Multi-language support (en, zh, zh-TW, ko, ja)
- Persona selector, layer legend, filter panel, search bar
- Mobile-responsive with bottom nav

**Hooks** (hooks/hooks.json):
- PostToolUse: Triggers on git commit/merge/rebase — checks config for autoUpdate and prompts incremental graph update
- SessionStart: Compares git commit hash from meta.json against HEAD — detects staleness and prompts update

### Key techniques

**Three-tier LLM output sanitization**: The schema validation pipeline is unusually defense-in-depth for LLM output. Tier 1 (sanitize): null→empty array, lowercase normalization. Tier 2 (auto-fix): missing field defaults, alias resolution (e.g., "low"→"simple", "uses"→"depends_on"), weight clamping, string-to-number coercion. Tier 3 (validate): Zod schema validation with referential integrity (dangling edges dropped). This is pragmatic engineering — treating LLM output as untrusted user input requiring the same validation rigor.

**Two-phase agent design**: Every agent follows a script-first pattern: a deterministic Node.js script handles the computationally expensive structural work (file discovery, tree-sitter parsing, import adjacency matrices), then the LLM handles only semantic interpretation (summaries, tags, complexity ratings, layer naming). This constrains the LLM to judgment tasks where it excels while keeping factual/structural data deterministic.

**Fingerprint-based incremental updates**: Rather than re-analyzing the entire codebase on every change, the system computes file fingerprints combining SHA-256 content hashes with structural signatures (function/class/import/export signatures). Changes are classified as NONE (identical), COSMETIC (content changed, structure identical), or STRUCTURAL (signatures changed). The change classifier then decides the update scope: skip, partial update of changed files only, architecture re-analysis, or full rebuild. The thresholds are concrete: >30 files or >50% of project triggers full rebuild; new/deleted directories or >10 files triggers architecture update.

**Import resolution across 12 languages**: The project-scanner agent specifies detailed import resolution algorithms for TypeScript/JS (relative + path alias), Python (both relative and absolute with __init__.py probing), Go (module path matching), Rust (crate/super/mod), Java/Kotlin (package→path mapping), Ruby (require_relative + load-path), PHP (PSR-4 namespace resolution), C/C++ (#include with include/src path probing). This is one of the most comprehensive cross-language import resolvers I've seen in an open-source code analysis tool.

**Dual layer detection**: The architecture analyzer uses both heuristic pattern matching (directory names like "routes"→API, "models"→Data) and LLM-based detection. The heuristic pass is always run first as a script, then the LLM can refine. The heuristic's directory-to-layer mapping covers 50+ patterns across multiple language ecosystems (including Python-specific patterns like `blueprints`, `serializers`, `templatetags`).

**Edge weight as ordering mechanism**: In the domain graph, `flow_step` edges use fractional weights to encode step ordering — monotonically increasing weights within 0-1 let consumers recover execution order without a separate ordering field.

**Absolute path sanitization**: The persistence layer automatically scrubs absolute file paths from knowledge-graph.json before writing to disk, preventing developer directory structure from leaking into committed output.

### Design decisions

**Optimized for deterministic correctness in structural analysis, LLM flexibility in semantics**: The split is deliberate and well-executed. Tree-sitter handles everything deterministic (function extraction, import parsing, line counting). The LLM handles everything interpretive (what does this file do, how complex is it, what tags apply). This is the right split — neither component alone could produce a quality graph.

**Sacrificed: analysis speed for thoroughness**: The full pipeline runs multiple agents sequentially with multiple LLM calls. Each file gets individual LLM analysis for summaries. This is expensive in both time and tokens but produces higher-quality output than a single-pass approach.

**Trade-off: comprehensive JSON artifact over direct LLM context**: Unlike tools that inject codebase context directly into LLM prompts, Understand-Anything produces a standalone JSON knowledge graph. This enables the dashboard, incremental updates, and reuse across sessions, but means the graph is a lossy representation of the codebase.

**LLM output is treated as adversarial**: The alias normalization tables (75+ node type aliases, 80+ edge type aliases, 6 complexity aliases, 5 direction aliases) reveal hard-won experience with LLM output inconsistency. Rather than begging the LLM to use canonical values (which prompt engineering can't fully guarantee), the system just accepts common aliases and maps them deterministically.

**Weakness: no code-level call graph**: The system extracts import relationships deterministically but call edges (`calls`) are LLM-inferred ("infer from imports + function names when confident"). There's no tree-sitter-level call graph extraction despite tree-sitter being available. This means call edges are less reliable than import edges.

**Weakness: the dashboard is read-only**: The dashboard visualizes the graph but doesn't allow editing. There's no feedback loop from human correction back into the graph — corrections would need to happen via direct JSON editing or re-running analysis.

### Comparison notes

**vs. DeepWiki**: DeepWiki generates on-demand AI-powered codebase explanations with line-level citations. Understand-Anything produces a persistent, structured knowledge graph that lives with the repo and updates incrementally. DeepWiki is better for one-shot exploration; Understand-Anything is better for ongoing orientation.

**vs. Trailmark**: Trailmark (Trail of Bits) treats source code as a queryable graph for security analysis. Understand-Anything's graph is more comprehensive (includes configs, docs, infra) but less security-focused. Both use graph representations but for different purposes.

**vs. GraphRAG**: GraphRAG (Microsoft) builds knowledge graphs from text for RAG applications. Understand-Anything builds them from code for developer orientation. The graph structures differ — GraphRAG focuses on entity-relationship-community hierarchies; Understand-Anything focuses on file-function-class containment and import/call relationships.

**vs. lat.md**: lat.md produces a markdown knowledge graph for codebases with drift validation. Understand-Anything produces JSON with a richer schema (35 edge types, 21 node types) and an interactive dashboard, but lat.md's markdown-native approach is more git-diff friendly.

**vs. graphify**: graphify produces multimodal knowledge graphs (code, PDFs, screenshots, diagrams). Understand-Anything is code-only but much deeper in analysis — graphify's strength is breadth of source types.

### Data flow

```
git ls-files → project-scanner → scan-result.json
                                     ↓
                        file-analyzer → batch-*.json
                                     ↓
                        merge-batch-graphs.py → assembled-graph.json
                                     ↓
                        assemble-reviewer → reviewed graph
                                     ↓
                        architecture-analyzer → layers.json
                                     ↓
                        domain-analyzer → domain-graph.json
                                     ↓
                        tour-builder → tour steps
                                     ↓
                        graph-reviewer → final knowledge-graph.json
                                     ↓
                        Dashboard renders interactive visualization
```

Auto-update flow:
```
git commit → PostToolUse hook → staleness check → change classifier
  → PARTIAL_UPDATE: re-analyze changed files
  → ARCHITECTURE_UPDATE: re-analyze + re-run architecture and tour
  → FULL_UPDATE: full re-analysis
```

### Tech stack

- TypeScript (core library + dashboard)
- tree-sitter (10 languages via web-tree-sitter)
- Zod v4 (schema validation)
- Fuse.js v7 (fuzzy search)
- ignore v7 (.gitignore pattern matching)
- yaml v2 (YAML parsing)
- React + ReactFlow (dashboard)
- ELK.js (graph layout)
- Vitest (testing)
