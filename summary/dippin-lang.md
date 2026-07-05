---
url: https://github.com/2389-research/dippin-lang
title: "Dippin: A Domain-Specific Language for AI Agent Workflows"
author: 2389 Research
date_fetched: 2026-05-31
date_published: 2025
---

# Dippin Language — Full Architectural Analysis

## Project Overview

Dippin is a domain-specific language (DSL) for defining AI agent workflow pipelines. It uses an indentation-sensitive syntax (like Python) to define DAGs of agent, human, tool, parallel, fan_in, subgraph, conditional, and manager_loop nodes, connected by edges with conditional routing. The language compiles to a canonical intermediate representation (IR) that serves as the contract between parsing and execution. The runtime execution engine ("tracker") is a separate project.

The project is written in Go (~328K lines of code across source and tests), with editor integration for Zed, VSCode, and tree-sitter. It provides a full toolchain: parser, validator, linter, formatter, simulator, cost estimator, optimizer, health doctor, graph renderer, coverage analyzer, dead-code detector, semantic differ, test runner, LSP server, and bundle packer/unpacker.

## Architecture

### Pipeline Architecture

Dippin follows a **classic compiler pipeline**:

```
.dip source → Lexer → Parser → IR → [Validator, Linter, Formatter, Simulator, ...]
```

The IR (`ir` package) is the single source of truth. All downstream tools consume the IR, not raw source. This is a clean separation that enables the rich tooling ecosystem — each tool is a pure function from `ir.Workflow → Result`.

### Core Packages

| Package | Role | Key Files |
|---------|------|-----------|
| `ir/` | Canonical intermediate representation | `ir.go` (types), `edge.go` (conditions), `source.go`, `lookup.go` |
| `parser/` | Lexer + recursive-descent parser | `parser.go`, `lexer.go` |
| `validator/` | Graph-structure validation (DIP001–DIP009) + semantic linting | `validate.go` (9 structural checks), `lint*.go` (~20 lint rules) |
| `simulate/` | Reference executor: walks the graph without side effects, emits JSONL events | `simulate.go`, `condition.go` (condition evaluator), `path_enumerator.go` (all-paths DFS), `parallel.go`, `events.go`, `defaults.go`, `interactive.go` |
| `cost/` | Heuristic cost estimation based on model pricing tables | `cost.go`, `pricing.go` |
| `optimize/` | Rule-based model selection optimization | `optimize.go` (4 rules), `prompt.go` |
| `doctor/` | Health report card (grade A–F) aggregating validator + coverage + cost | `doctor.go` |
| `coverage/` | Edge coverage and reachability analysis |
| `formatter/` | Canonical .dip formatter (2-space indent) |
| `lsp/` | Language Server Protocol implementation | `server.go`, `diagnostics.go`, `completion.go`, `hover.go`, `definition.go`, `symbols.go` |
| `graph/` | ASCII DAG renderer | `render.go` |
| `diff/` | Semantic diff between two workflows |
| `unused/` | Dead-branch detection |
| `dipx/` | Bundle format (tar.gz-based) with manifest |
| `export/` | Export flattened workflow as .dip |
| `flatten/` | Subgraph inlining |
| `event/` | Canonical JSONL event protocol |
| `migrate/` | DOT-to-.dip migration |
| `testrunner/` | Test runner for .test.json test files |
| `editors/` | VSCode extension, Zed extension, tree-sitter grammar |

### Key Types (IR)

The `ir.Workflow` struct is the top-level type. Key design decisions:

1. **Sealed NodeConfig interface**: Each node kind has its own config type implementing `NodeConfig`. The sealed interface prevents invalid node kind/config combinations structurally (a Go interface with unexported method).

2. **Explicit start/exit**: Every workflow must declare explicit entry and exit node IDs. This is validated as DIP001/DIP002. Unlike many workflow systems that infer start/exit, Dippin requires them explicitly — the author's intent must be unambiguous.

3. **Lazy condition parsing**: Edge conditions are stored as raw strings in the IR but lazily parsed into AST form (`CondCompare`, `CondAnd`, `CondOr`, `CondNot`) by the simulator/validator. This keeps the parser fast and avoids parsing conditions that aren't needed.

4. **SourceMap**: Every node and edge carries a `SourceLocation` (file, line, column) for precise diagnostics.

## Key Techniques

### 1. Indentation-Sensitive Lexer

The lexer emits `INDENT`/`OUTDENT` tokens (like Python) rather than relying on braces. This is a deliberately human-friendly choice — the `.dip` format is meant to be authored by humans, not just machines. The trade-off is parser complexity (indentation tracking in the lexer) vs. a more readable format than DOT/Graphviz.

### 2. DSL-as-Compiler Architecture

Rather than being "just another YAML config," Dippin is a full compiler with IR, validation passes, and a clear separation between frontend (parser), middle-end (IR, validator), and backend (formatter, simulator). This is unusual for a workflow DSL — most stop at "parse YAML → execute."

### 3. Reference Simulator (Zero Side-Effects)

The simulator (`simulate/`) walks the IR graph without making real LLM calls or running real commands. It emits JSONL events that mirror exactly what the real executor would emit. This enables:
- **Scenario testing**: `--scenario outcome=fail` lets you test error paths without running expensive models
- **Path enumeration**: `--all-paths` does a DFS of all possible execution paths (bounded at 100 results, 200 depth)
- **Conditional seeding**: In all-paths mode, the `seedConditionContext` function sets context values to satisfy each branch's conditions

### 4. Lazy AST Construction for Conditions

Edge conditions are tokenized and parsed into an AST on demand (not during initial parsing). The `simulate/condition.go` file contains a complete recursive-descent parser for boolean expressions:
```
expr = orExpr
orExpr = andExpr ("or" andExpr)*
andExpr = unary ("and" unary)*
unary = "not" unary | compare
compare = VARIABLE OP VALUE
```
This supports infix `not` ("variable not contains value"), which is syntactic sugar for `not variable contains value`.

### 5. Structural Validation with "Did You Mean?"

The validator (`validator/validate.go`) implements 9 structural checks (DIP001-DIP009) including cycle detection via DFS white/gray/black coloring, reachability via BFS, and parallel/fan-in pairing. For unknown node references, it computes Levenshtein distance ≤ 2 to suggest corrections ("did you mean X?").

### 6. Adversarial Edge Resolution

The simulator's `resolveNext` method implements a priority-based edge resolution:
1. Single unconditional edge → take it
2. Human node with labeled edges → match `preferred_label` from context
3. MaxNodeVisits exceeded → force loop-exit (first non-matching conditional)
4. Conditional edges in declaration order → first match wins
5. Fallback to first unconditional edge (default path)
6. If no match at all → first edge (happy-path default)

This is a deliberate design: the simulator does NOT try all possible conditional matches (that's `--all-paths`). The single-path mode mimics what a real executor would do with actual LLM outputs.

### 7. Bundle Format (.dipx)

The `dipx/` package implements a tar.gz-based bundle format that packages a .dip entry point with all its subgraph references. It uses manifest-based integrity verification. This solves the distribution problem: you can't ship a .dip file that references subgraph files without also shipping those files.

### 8. Heuristic Cost Model

The cost estimator (`cost/cost.go`) uses heuristic token counts based on prompt length and model-tier-specific output estimates (Opus: 1500 tokens/turn, Sonnet: 1000, Haiku: 400). It also factors in loop multipliers for nodes targeted by restart edges. The pricing table is configurable.

## Design Decisions

### Optimized for: Human Authorship over Machine Generation

The indentation-sensitive syntax, multiline blocks (no quoting needed), and section headers as comments are all designed for humans writing .dip files. This is the opposite of DOT/Graphviz, which is designed for machine generation and painful to write by hand.

### Optimized for: Tooling Depth over Runtime Simplicity

The project invests heavily in static analysis tools (validator, linter, optimizer, doctor, coverage, unused) rather than in the runtime executor. The runtime ("tracker") is a separate project. This means the language itself can evolve and be reasoned about independently of execution semantics.

### Trade-off: String-Only Conditions

All condition comparisons are string-based — there is no numeric coercion. `ctx.score = 10` is a string comparison. This simplifies the condition model but means workflows can't express numeric comparisons (>, <, >=). The syntax deliberately avoids these.

### Trade-off: No Backend Bundled

The actual LLM execution engine is not in this repo. Dippin is "just" the language and tooling. This is a clean separation (language vs. runtime) but means you can't use dippin without also having the tracker.

### Trade-off: Advisory I/O Declarations (v1)

The `NodeIO` type (declared reads/writes per node) is explicitly marked as "advisory in v1 — validated as warnings, not errors." This means data flow between nodes is documented but not enforced, which is pragmatic for v1 but leaves correctness gaps.

## Comparison to Related Projects

- **vs. DOT/Graphviz**: DOT is a graph description language; Dippin is a workflow execution language. DOT has no notion of models, prompts, retries, or conditions. Dippin's syntax is more human-friendly (indentation vs. braces/semicolons).
- **vs. GitHub Actions / CI YAML**: Both are DAG-oriented pipeline languages, but Dippin is designed for LLM agents (with prompt management, model selection, cost estimation), not CI/CD.
- **vs. LangChain/LangGraph**: LangGraph is a Python library for building agent graphs; Dippin is a standalone language with its own toolchain. Dippin is more declarative and toolable.
- **vs. [[workgraph]]**: Both represent agent tasks as graphs, but workgraph is a runtime system (JSONL on disk, agents come and go) while Dippin is a language definition with a separate runtime.
- **vs. [[Cord]]**: Cord has spawn/fork/ask primitives for dynamic task trees; Dippin is a static DAG with restart edges for loops.

## Why This Matters

Dippin represents a bet that **agent workflow definitions should be first-class artifacts** — version-controlled, lintable, optimizable, and human-authored. It treats workflow definitions the way compilers treat source code: parse, validate, optimize, execute. The separation of language from runtime is particularly clever — it means the same .dip file could target different executors, and the tooling investment is amortized across all of them.

The project's depth (24 CLI subcommands, LSP server, 9 structural validators, 20+ lint rules, 4 optimization rules, tree-sitter grammar, editor extensions) shows serious engineering investment. This isn't a weekend project — it's a language platform.
