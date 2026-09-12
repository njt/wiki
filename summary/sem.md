---
title: "sem — Semantic Version Control"
url: https://github.com/Ataraxy-Labs/sem
author: Ataraxy Labs
date_fetched: 2026-06-12
date_published: 2025-06-01
topics:
  - developer-tools
---

# sem: Deep Architectural Analysis

sem is a Rust workspace (~55K LoC) that layers entity-level version control on top of Git. Instead of "lines x-y changed," it tells you "function `authenticateUser` was modified and `validateToken` was renamed." Built by Ataraxy Labs as agent-native infrastructure for software development.

## Repository Structure

```
crates/
  sem-core/     — engine: entity extraction, dependency graph, scope resolution
  sem-cli/      — CLI binary (sem diff, sem impact, sem blame, etc.)
  sem-mcp/      — MCP server exposing 6 tools to AI agents
  sem-plugin/   — WASM plugin system for community-contributed language parsers
```

Plus JS wrapper (npm package `@ataraxy-labs/sem`), Dockerfile, Homebrew formula, Nix flake.

## Core Architecture

### Entity Model (`crates/sem-core/src/model/`)

**SemanticEntity** (`entity.rs`) is the fundamental type. Every function, class, method, struct, or trait is represented as:
- `id`: `"src/auth.ts::function::validateToken"` — a deterministic compound key
- `content_hash`: xxhash of the raw text
- `structural_hash`: AST-based hash that strips comments and normalizes whitespace (inspired by Unison's content-addressed model). Two entities with the same structural_hash are logically identical even if formatting differs.
- `parent_id`: links methods to their parent class, nested functions to their enclosing function

Entity IDs support line-number disambiguation (`@L42`) and same-line ordinal disambiguation (`#2`) for overloaded/duplicate names.

**ChangeType** is a 6-variant enum: Added, Modified, Deleted, Moved, Renamed, Reordered. "Reordered" is particularly notable — it detects when entities moved within a file but didn't change content, using a longest-non-decreasing-subsequence algorithm on the "after" positions.

### Entity Matching — 5-Phase Algorithm (`identity.rs`, ~1474 lines)

This is the heart of the differ. The algorithm runs in 5 phases:

**Phase 1: Exact ID match.** Same entity ID in before/after → content hash comparison → Modified or unchanged.

**Phase 2: Content hash match.** Different IDs but same content hash → rename or move detection. Falls back to structural_hash (formatting/comment changes don't count).

**Phase 3: Same-file signature match.** Same (file, entity_type, name, parent) but different IDs → matches overloaded names whose collision group size changed. Uses Jaccard similarity with a 30% floor, then picks best by similarity score + line distance.

**Phase 4: Cross-file signature match.** Same (entity_type, name, parent) across different files → detects moves/renames when the file itself was renamed. Critical for avoiding false add/delete pairs on file renames.

**Phase 5: Fuzzy similarity.** Jaccard index on whitespace-split tokens at >80% threshold for remaining unmatched entities. Groups by entity type to avoid O(N×M). Has a size-ratio cutoff at 50% to skip obviously unrelated pairs early.

**Phase 6: Intra-file reorder detection.** For entities matched by exact ID with identical content, checks if relative ordering changed using a longest-non-decreasing-subsequence on after-positions. Entities outside the LNDS set are marked as Reordered.

The TokenCache lazily tokenizes content into `HashSet<&str>` for Jaccard computation, avoiding re-tokenization across phases.

### Dependency Graph (`graph.rs`, ~8791 lines — the largest file)

Two-pass approach inspired by arXiv:2601.08773 (Reliable Graph-RAG):

**Pass 1:** Extract all entities, build symbol table (name → Vec<entity_id>). Also builds Go package index (pkg_name → entities) for O(1) cross-file method resolution, and class_members/owner_members indexes.

**Pass 2:** For each entity, extract references and resolve against the symbol table. This is where things get interesting:

The system has TWO reference extraction strategies:
1. **Bag-of-words** (original): Regex-based extraction of identifier-like tokens from entity content, resolved against the symbol table. Simple but prone to false positives from same-name entities in different scopes.
2. **Scope-aware resolution** (`scope_resolve.rs`, ~6233 lines): Tree-sitter AST walk that builds scope chains, tracks variable types through assignments, and resolves references with compiler-like accuracy. Enabled per-language via `ScopeResolveConfig`.

The scope resolver implements:
- Nested scopes (module → class → function → block)
- Variable type tracking: `x = Foo()` → `x.method()` resolves to `Foo.method()`
- Pending call type resolution: `x = func()` where `func` returns a known type gets resolved in a second pass
- Import resolution across 8 import styles (JS/TS `from`, Python `import`, Go package imports, Rust `use`, etc.)
- Constructor parameter type inference: scans `__init__`/constructor bodies for `self.attr = param` patterns
- Swift/Kotlin external method handling (methods declared outside class bodies)

The scope resolver is parallelized per-file via rayon and includes a `ScopeLookupCache` to avoid repeated HashMap lookups in the hot per-reference loop.

### Parser Plugin System (`plugin.rs`, `registry.rs`)

Simple trait-based plugin system:

```rust
pub trait SemanticParserPlugin: Send + Sync {
    fn id(&self) -> &str;
    fn extensions(&self) -> &[&str];
    fn extract_entities(&self, content: &str, file_path: &str) -> Vec<SemanticEntity>;
    fn structural_hash_content(&self, content: &str, file_path: &str) -> Option<String>;
    fn compute_similarity(&self, a: &SemanticEntity, b: &SemanticEntity) -> f64;
}
```

ParserRegistry maps file extensions → plugins. Supports:
- Custom extension mappings via `.semrc` (project-local config)
- `.gitattributes` parsing for `diff=` and `linguist-language=` directives
- Content-based language detection for extensionless files (shebang, vim modeline, import/declaration patterns for 19 languages)
- Fallback plugin for unrecognized formats (chunk-based diffing)

The "code" plugin handles 28 programming languages via tree-sitter grammars, each with a language config specifying entity node types, scope resolution rules, assignment strategies, import extraction, etc. The config system is substantial: `languages.rs` (~2536 lines) defines per-language extraction and scope resolution rules.

### Diff Pipeline (`differ.rs`, ~1372 lines)

The top-level pipeline:
1. For each changed file, resolve the parser plugin
2. Extract before/after entities (with `catch_unwind` guards against parser panics)
3. Run the 5-phase entity matching
4. Suppress redundant parent changes (if a method changed inside a class, the class's "changed" status is explained by the child change)
5. Detect orphan changes (lines changed outside any entity span)
6. Aggregate counts and produce the DiffResult

All per-file processing is parallelized with rayon. The differ also handles binary files, stdin input (for pipe workflows), and unified diff patches.

### Impact Analysis (`commands/impact.rs`, ~613 lines)

Uses the dependency graph to answer "what breaks if X changes?":
- Direct dependencies / dependents
- Transitive impact (configurable depth, default 2)
- Test-only impact filtering (using test directory detection + naming conventions)
- JSON and terminal output formats

### Context Generation (`parser/context.rs`, ~487 lines)

Token-budgeted context for LLMs: given an entity name, gathers the entity's content + its dependencies + its dependents, trimmed to fit a strict token budget. When the target signature itself doesn't fit, JSON output reports `target_omitted: true`.

### MCP Server (`sem-mcp/`)

Exposes 6 tools mirroring the CLI commands: `sem_entities`, `sem_diff`, `sem_blame`, `sem_impact`, `sem_log`, `sem_context`. Uses stdio transport. The server maintains an in-memory SQLite-backed entity cache (`cache.rs`, ~2137 lines) to avoid re-parsing every request.

### Git Integration (`git/`)

Uses `git2` (libgit2 bindings) for all Git operations. The bridge module (~1953 lines) handles:
- Diffing between commits, working tree, index
- File status detection (added, deleted, modified, renamed)
- Binary file detection
- Jujutsu (jj) VCS support via the `jj` module

### Performance Engineering

- **mimalloc** allocator for both CLI and MCP binaries
- **rayon** parallel iterators with feature-gated `maybe_par_iter!` macro (compiles to sequential when `parallel` feature disabled)
- **SQLite entity cache** stored in OS cache dir to avoid re-parsing
- **xxhash (XXH3)** for content and structural hashing
- **LTO + single codegen unit** in release profile with stripping
- **Grammar feature flags** allowing compilation with only needed language parsers (`grammar-core` = 6 languages, `grammar-all` = all 31)
- `thin` LTO in release, `fat` LTO for WASM builds

### Design Trade-offs

**Optimized for:** speed (parallel per-file processing, caching), correctness of entity matching (5-phase algorithm with structural hashing), agent consumption (JSON output, MCP server, token-budgeted context).

**Sacrificed:** full type-level analysis (it's scope-based, not type-checker level — will resolve `x.method()` to `Foo.method()` through type tracking but won't catch generics), memory efficiency (the dependency graph is built in memory, ~55K LoC suggests significant memory use on large repos).

**Clever insight:** The structural_hash concept (inspired by Unison) is the killer feature. By computing an AST-normalized hash that strips comments and whitespace, sem distinguishes cosmetic changes from logic changes without needing to understand the semantics. This is a practical middle ground between "all changes are equal" (line diff) and "full semantic understanding" (compiler-level).

**Weakness:** Scope resolution is language-specific and requires per-language configuration. The scope_resolve config system has grown organically and has substantial duplication across languages. The fallback to bag-of-words for unconfigured languages produces lower-quality dependency graphs.

**Surprise:** The reorder detection is unusually sophisticated for a diff tool. Using LNDS to find the minimum set of moved entities means you don't get 100 "reordered" entries when someone alphabetizes a file — you get the minimum explanation.

### Comparison to Related Tools

- **tree-sitter itself**: sem uses tree-sitter for parsing but adds entity extraction, matching, and dependency analysis on top
- **GitHub's stack graphs / code navigation**: Similar semantic indexing, but sem works locally without a server and is designed for diff/impact rather than navigation
- **Sourcegraph / lsif**: Server-side semantic indexing; sem is local-first and works on any Git repo with no setup
- **difftastic**: Also uses tree-sitter for structural diffs, but focuses on presentation (syntax-aware coloring) rather than entity-level change tracking

### Languages and Plugins

31 programming languages: TypeScript, JavaScript, Python, Go, Rust, Java, C, C++, C#, Ruby, PHP, Swift, Elixir, Bash, HCL/Terraform, Kotlin, Fortran, Vue, XML, ERB, Svelte, Perl, Dart, OCaml, Scala, Nix, Haskell, Elm, Clojure, D, Zig

6 structured data formats: JSON, YAML, TOML, EDN, CSV, Markdown

Plus fallback chunk-based diffing for everything else.

### Dependencies

- tree-sitter (native Rust bindings, ~30 grammar crates as optional deps)
- git2 (libgit2)
- rayon (parallel iterators)
- xxhash-rust (XXH3 hashing)
- serde/serde_json/serde_yaml/toml (serialization)
- regex (import resolution and test detection)
- mimalloc (allocator)
- clap (CLI argument parsing)
- colored (terminal output)
