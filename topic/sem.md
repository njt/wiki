# sem

Semantic version control that diffs at the entity level — functions, classes, methods — not lines. Built by Ataraxy Labs as agent-native infrastructure: every command has JSON output, an MCP server exposes 6 tools to AI agents, and `sem context` generates token-budgeted LLM summaries. 31 languages via tree-sitter, works in any Git repo with no setup.

#tool #version-control #semantic-analysis #tree-sitter #agent-tooling #rust

---

## Architecture

A Rust workspace (~55K LoC) in four crates:

- **sem-core** — the engine: entity extraction, dependency graph building, scope-aware reference resolution, and the 5-phase entity matching algorithm
- **sem-cli** — the CLI binary: `sem diff`, `sem impact`, `sem blame`, `sem log`, `sem entities`, `sem context`, `sem verify`, `sem graph`
- **sem-mcp** — MCP server exposing the same 6 commands as tools via stdio transport, with an in-memory SQLite entity cache
- **sem-plugin** — WASM plugin system for community-contributed language parsers

An npm wrapper (`@ataraxy-labs/sem`) downloads the binary at install time so JS/TS projects can add it as a devDependency.

### Entity Model

Everything revolves around `SemanticEntity` (`crates/sem-core/src/model/entity.rs`). Each entity has:
- A deterministic compound ID: `"src/auth.ts::function::validateToken"` with optional line-number (`@L42`) and ordinal (`#2`) disambiguators for overloads
- `content_hash` (xxhash of raw text) and `structural_hash` (AST-normalized hash that strips comments/whitespace — inspired by Unison's content-addressed model)
- `parent_id` linking methods to their parent class, nested functions to their enclosing scope

`ChangeType` is a 6-variant enum: **Added, Modified, Deleted, Moved, Renamed, Reordered**. The "Reordered" variant is unusual — it detects when entities shifted position within a file without changing content.

### Entity Matching — 5 Passes

The core algorithm in `identity.rs` (~1474 lines) runs in five phases, each handling what the previous couldn't:

1. **Exact ID match** — same entity ID in before/after → compare content hash → Modified or unchanged
2. **Content hash match** — different ID but same hash → Renamed or Moved. Falls back to structural_hash (so reformatting doesn't show as a change)
3. **Same-file signature match** — same (file, entity_type, name, parent) but different IDs → handles overload collision groups that grew or shrank. Uses Jaccard similarity with a 30% floor
4. **Cross-file signature match** — same (entity_type, name, parent) across different files → catches moves/renames when the file itself was renamed. Prevents false add/delete pairs on file renames
5. **Fuzzy Jaccard similarity** — >80% token overlap for remaining unmatched. Groups by entity type to avoid O(N×M). Size-ratio cutoff at 50% skips obviously unrelated pairs early

Plus a final **reorder detection** pass: unchanged entities whose relative order shifted are identified via longest-non-decreasing-subsequence on after-positions — you get the minimum set of "reordered" entries, not one per entity.

A `TokenCache` lazily tokenizes entity content into `HashSet<&str>` for Jaccard, avoiding re-tokenization across phases.

### Dependency Graph

`graph.rs` (~8791 lines) builds a cross-file dependency graph in two passes:

**Pass 1:** Extract all entities, build symbol table (name → Vec<entity_id>), Go package index (pkg_name → entities for O(1) cross-file method resolution), class_members and owner_members indexes.

**Pass 2:** For each entity, extract references and resolve against the symbol table. This is where sem diverges from simpler tools — it has **two reference extraction strategies**:

- **Bag-of-words** (original): Regex extraction of identifier-like tokens. Fast but produces false positives from same-name entities in different scopes.
- **Scope-aware resolution** (`scope_resolve.rs`, ~6233 lines): Walks tree-sitter ASTs to build scope chains (module → class → function → block), tracks variable types through assignments (`x = Foo()` → `x.method()` resolves to `Foo.method()`), and resolves references with compiler-like accuracy. Enabled per-language via `ScopeResolveConfig`.

The scope resolver's type tracking is surprisingly sophisticated for a diff tool: it scans `__init__`/constructor bodies for `self.attr = param` patterns, infers constructor parameter types from call sites, and resolves pending call types (`x = func()` where `func` returns a known type) in a second pass.

### Parser Plugin System

`ParserRegistry` maps file extensions to plugins implementing the `SemanticParserPlugin` trait. Supports:
- Custom mappings via `.semrc` (project-local: `.mypy = python`)
- `.gitattributes` parsing for `diff=` and `linguist-language=`
- Content-based detection for extensionless files (shebang, vim modeline, import/declaration patterns for 19 languages)
- Fallback plugin doing chunk-based diffing for unrecognized formats

The "code" plugin handles 28 languages via tree-sitter grammars, each with a config in `languages.rs` (~2536 lines) specifying entity node types, scope resolution rules, assignment strategies, and import extraction patterns.

### Key Commands

| Command | What it does |
|---------|-------------|
| `sem diff` | Entity-level diff with rename/move/reorder detection, 5 output formats (terminal, json, markdown, plain, git-style) |
| `sem impact` | "What breaks if X changes?" — direct deps, dependents, transitive (configurable depth), test-only filtering |
| `sem blame` | Per-entity git blame — who last modified each function |
| `sem log` | Evolution of a single entity through git history, with optional content diffs between versions |
| `sem entities` | List all entities under a path |
| `sem context` | Token-budgeted LLM context: entity + deps + dependents fitted to a budget. Reports `target_omitted: true` when the target signature itself doesn't fit |
| `sem verify` | Function call arity checking across the codebase — finds broken callers from signature changes |
| `sem setup` | Replaces `git diff` output with `sem diff` globally via git config |

---

## Key Techniques

### Structural Hashing

The killer feature. By computing an AST-normalized hash that strips comments and normalizes whitespace, sem distinguishes "the logic changed" from "someone ran the formatter." This is a pragmatic middle ground between line-level diffing (no semantic understanding) and compiler-level analysis (too heavy). Inspired by Unison's content-addressed code model, though sem applies it at the entity level rather than the definition level.

### Scope-Aware Reference Resolution

Instead of treating all references as bag-of-words token matches, the scope resolver walks tree-sitter ASTs to build actual scope trees. It uses an iterative worklist (not recursion) to avoid stack overflow on deeply nested ASTs — a fix for [issue #103](https://github.com/Ataraxy-Labs/sem/issues/103). The per-reference loop hoists HashMap lookups out via a `ScopeLookupCache`, and entity ownership of references uses pre-computed byte spans rather than repeated hash lookups.

### Type Tracking Through Assignments

The scope resolver records type bindings from assignments (`x = Foo()` → types["x"] = "Foo"), then uses those bindings to resolve method calls (`x.method()` → Foo.method()). This works across Python, TypeScript, Rust, Go, Swift, and Kotlin with per-language assignment strategies (`AssignmentStrategy` enum: LeftRight, Declarators, PatternBased, ShortVar, VarSpec).

### LNDS-Based Reorder Detection

When entities with identical content shift position within a file, sem uses a longest-non-decreasing-subsequence algorithm to find the minimum set of reordered entities. This means alphabetizing 50 functions doesn't produce 50 "reordered" entries — you get the handful that actually moved relative to each other.

### Parallel Architecture

Everything per-file is parallelized via rayon with a `maybe_par_iter!` macro that compiles to sequential iteration when the `parallel` feature is disabled. The scope resolver processes files in parallel in both Pass 1 (return type scanning) and Pass 2 (reference resolution), with results merged sequentially.

---

## Design Decisions

### Optimized for Agent Consumption

Every CLI command has `--json` output. The MCP server exposes the same 6 tools agents need. `sem context` generates budget-constrained summaries. This isn't a diff tool that happens to output JSON — it was architected for machine consumers from the start.

### Caching Strategy

Entity data is cached in SQLite stored in the OS cache directory (not the repo). Cache location is outside the working tree so cache files don't dirty `git status`. Set `SEM_CACHE_DIR` to override. The MCP server maintains its own in-memory cache for sub-millisecond response times.

### Grammar Feature Flags

Each tree-sitter grammar is an optional Cargo feature. You can compile with just `grammar-core` (6 languages) or `grammar-all` (31). This matters for the WASM plugin build and for users who only need a few languages.

### What's Sacrificed

- **Full type resolution**: Scope tracking catches `x = Foo(); x.method()` but won't resolve generics, traits, or interfaces. It's scope-level, not type-checker level.
- **Memory**: The dependency graph is built entirely in memory. On very large repos (100K+ files), this could be a problem. The SQLite cache mitigates re-parse cost but not graph memory.
- **Extensibility**: Adding a new language requires writing tree-sitter queries + Rust plugin code. The WASM plugin system exists but is nascent.

### The `structural_hash` Insight

This is the design decision that makes the whole thing work. Content hash alone is too brittle (formatter runs show every entity as modified). AST comparison alone is too expensive. The structural hash — computed once, stored, compared as a cheap string — gives you "did the logic change?" for O(1) comparison cost per entity pair.

---

## Comparison Notes

**vs. difftastic**: Both use tree-sitter. difftastic focuses on syntax-aware rendering (pretty colors for structural changes); sem focuses on entity-level change tracking (what changed, not how to display it). difftastic is a better *viewer*; sem is a better *analyzer*.

**vs. GitHub stack graphs / code navigation**: Both do semantic indexing. GitHub's is server-side with a complex ingestion pipeline; sem is local, instant, and designed for diff/impact rather than navigation.

**vs. Sourcegraph / LSIF**: Server-side semantic indexing with language servers. sem sacrifices full language server accuracy for zero-config operation on any Git repo. The scope resolver is a middle ground — compiler-like but not compiler-level.

**vs. [[graphify]]**: graphify builds multimodal knowledge graphs from codebases (code + PDFs + diagrams + screenshots). sem is focused on entity-level diff and impact analysis within the code alone. Complementary: sem tells you what entities changed and what depends on them; graphify tells you how everything connects.

**vs. [[Understand-Anything]]**: Both use tree-sitter + LLM pipelines. Understand-Anything builds persistent knowledge graphs with incremental git-hook updates. sem is lighter weight — no LLM, no persistence beyond the SQLite entity cache, instant results.

**vs. [[Git Diff Drivers]]**: git's external diff driver interface is the mechanism; sem is one implementation of that interface (via `sem setup`). sem goes far beyond what a simple diff driver can do by building and caching a full dependency graph.

**vs. [[lat.md]]**: Both produce navigable codebase knowledge. lat.md is markdown-based and validation-focused. sem is CLI-based and diff/impact-focused. Different use cases: lat.md for understanding a codebase's structure; sem for understanding what changed and what breaks.

---

*Sources: [[summary/sem]]*
*Last updated: 2026-06-12*
