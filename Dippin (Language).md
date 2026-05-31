# Dippin (Language)

Dippin is a domain-specific language for defining AI agent workflow pipelines. It uses Python-like indentation syntax to declare DAGs of agent, human, tool, and orchestration nodes, connected by conditional edges. The language compiles to a canonical IR and comes with a rich toolchain: parser, validator, linter, formatter, simulator, cost estimator, optimizer, health doctor, LSP server, and bundle packer. The runtime execution engine ("tracker") is separate — Dippin is purely the language definition and static analysis platform.

---

## Architecture

Dippin follows a **compiler pipeline**: `.dip` source → lexer → parser → IR → downstream tools. The IR (`ir/ir.go`) is the contract — every tool consumes the IR, not raw source.

The IR defines 8 node kinds, each with a sealed config type:

- **agent** (`ir.AgentConfig`): LLM call with prompt, model, provider, max_turns, response_format, tool_access
- **human** (`ir.HumanConfig`): Interactive gate with choice/freeform/interview/yes_no modes
- **tool** (`ir.ToolConfig`): Shell command execution with timeout, output declarations, marker grep
- **parallel** (`ir.ParallelConfig`): Fan-out to multiple target nodes
- **fan_in** (`ir.FanInConfig`): Join point collecting from source nodes
- **subgraph** (`ir.SubgraphConfig`): Embedded sub-workflow with parameter overrides
- **conditional** (`ir.ConditionalConfig`): Pure routing node, no LLM call
- **manager_loop** (`ir.ManagerLoopConfig`): Supervisor that polls and steers a child subgraph

Edges carry conditions (`ir.Condition` with AST), labels, weights, and a `restart: true` flag for loop back-edges. Conditions support `and`/`or`/`not` with string comparison operators (`=`, `!=`, `contains`, `startswith`, `endswith`, `in`).

The 24 CLI subcommands (in `cmd/dippin/cli.go`) map almost 1:1 onto packages, making the tool a thin CLI shell over library code.

## Key Techniques

**Indentation-sensitive lexer.** The lexer emits `INDENT`/`OUTDENT` tokens like Python. This was a deliberate choice: .dip files are meant for human authorship, not machine generation. The canonical formatter always outputs 2-space indentation.

**Lazy condition parsing.** Edge conditions are stored as raw strings in the IR and parsed into an AST (`simulate/condition.go`) only when needed. The condition parser is a recursive-descent implementation supporting infix `not` syntax: `ctx.outcome not contains "error"`.

**Priority-based edge resolution.** The simulator resolves edges in a specific order: single unconditional → human label matching → conditional in declaration order → unconditional fallback → happy-path default. The `--all-paths` flag switches to a DFS path enumerator that explores every branch, seeding context values to satisfy each condition.

**Structural validation with suggestions.** The validator (`validator/validate.go`) implements 9 DIP checks including DFS cycle detection (white/gray/black coloring) and BFS reachability. When it finds a dangling edge reference, it computes Levenshtein distance to suggest corrections.

**Heuristic cost estimation.** The cost package estimates per-node cost using prompt length ÷ 4 for input tokens and model-tier-specific output estimates (Opus: 1500/turn, Sonnet: 1000, Haiku: 400). Loop multipliers scale cost for nodes targeted by restart edges.

**Lazy-first toolchain design.** Rather than making the runtime smarter, Dippin invests in static analysis: 9 structural validators, 20+ lint rules, 4 optimization rules, dead-code detection, coverage analysis, and a health report card (grade A–F). This follows the [[Guardrails and Feedback Loops]] pattern — deterministically enforce correctness before execution.

## Design Decisions

**Human authorship over machine generation.** The indentation syntax, multiline blocks (no quoting), and section headers as comments are all optimized for humans writing .dip files — the opposite of DOT/Graphviz, which is designed for machine generation and is painful to write by hand.

**Language separate from runtime.** Dippin defines the language and tooling; the execution engine ("tracker") is a separate project. This means the .dip format could target different runtimes, and the static analysis investment is amortized across all of them. It also means the language can evolve independently of execution semantics.

**String-only conditions.** All comparisons are string-based — no numeric operators (`<`, `>`, `>=`). This simplifies the condition model but means workflows can't express numeric thresholds directly.

**Advisory I/O (v1).** Node-level reads/writes declarations are documented but not enforced — they produce warnings, not errors. Pragmatic for v1, but leaves data-flow correctness unverified.

**No runtime bundled.** You need the separate tracker project to actually execute a .dip workflow. Dippin is "just" the language — but it's an unusually thorough language definition.

## Comparison Notes

**vs. DOT/Graphviz:** DOT describes graphs; Dippin defines executable workflows with model selection, prompts, retries, and conditional routing. Dippin's indentation syntax is far more ergonomic for human authors. See [[The Dark Factory is a DOT File]] — Dippin may literally be the .dip file that replaces the DOT file.

**vs. CI/CD YAML:** Both are DAG pipelines, but Dippin targets LLM agent orchestration (prompt management, model cost estimation, checkpoint fidelity) rather than CI steps.

**vs. [[Cord]]:** Cord has dynamic spawn/fork/ask primitives for task trees; Dippin is a static DAG with restart edges for loops. Cord is a runtime; Dippin is a language definition.

**vs. [[workgraph]]:** Workgraph is a runtime system (JSONL on disk, agents come and go). Dippin's graph is defined upfront as source code and checked statically before execution.

**vs. [[Gas Town's Agent Patterns]]:** Gas Town uses ad-hoc orchestration scripts; Dippin formalizes the same idea into a typed, validated, toolable language. Dippin brings compiler-level static analysis to the problem Gas Town solves with bash.

**vs. [[Agent-Native Architectures (Every)]]:** Dippin embodies several of Every's five principles: parity (text format), granularity (nodes as composable units), and composability (subgraphs with parameter overrides). It's a concrete implementation of principles that Every describes in the abstract.

---

*Tags: #tool #project #agents #orchestration #dsl #language*

*Sources: [[raw/dippin-lang]]*
*Last updated: 2026-05-31*
