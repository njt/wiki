---
url: https://github.com/jagg-ix/zil-lean
title: ZIL Lean
author: jagg-ix
date_fetched: 2026-08-01
date_published: 2026
---

# ZIL Lean — Full Architecture Analysis

## Project Overview

ZIL (ZIL is a Language) is a small relational language for describing named objects, their relationships, and Horn-clause rules that derive additional relationships. ZIL Lean is a dual implementation: a Lean 4 formal DSL embedded directly in the proof assistant, and a Clojure/DataScript runtime that supports a `.zc` file format with macros, import/export tools, and adapters. The project uses relation tuples (`subject ── relation ──▶ object`) influenced by Google's Zanzibar authorization paper, applying that compact relational model to wider project relationships: requirements coverage, dependency tracking, change impact analysis, formal verification scheduling, and agent context handoff.

## Repository Structure

```
Zil/                 native Lean library (~70+ Lean files)
  Core/              fundamental types: Term, Atom, Rule, Query, Declaration, Program, Revision
  Datalog/           clause-logic surface: Eval, Model, Unify, Semantics, Environment
  Engine/            stratified Horn-rule evaluation, provenance tracing, query solving
  Syntax/            Lean-embedded DSL macros (zil_fact, zil_theorem_rule, zil_rule, zil_typed_rule)
  Parser/            .zc file parser, macro expander, declaration parser
  Codec/             deterministic encode/decode for relations, rules, conformance, revision
  Profile/           type profiles for typed rules
  Environment/       Lean environment extension for persistent facts/rules
  Exchange/          JSON-RPC-style worker protocol (ZIL-EXCHANGE/1)
  CLI/               native zil executable with ~20 subcommands
  Test/              ~25 test modules
  Generated/         Lean-generated safety proofs
examples/
  lean/              progressive native examples (01-06)
  native-cli/        CLI snapshot examples
  lean4-integration/  generated modules
src/zil/             Clojure runtime (~70+ Clojure files)
  core.clj           parser, evaluator, DataScript backend
  bridge/            Lean interop bridges (workflow, proof tokens, theorem CI, VStack)
  control/           capability model, command routing, durable event store
  runtime/           DataScript runtime, codec, ingest, REST/file/command adapters
  extensions/        SQLite event store, report exporter, external solver
  plugin/            plugin system with registry, evidence, manifests
  port/              conformance, native macro, library, retirement
lib/                 ~20 macro libraries (.zc files)
spec/                ~40 specification documents (v0.1 and v1)
tools/               import/export scripts (AWS, K8s, Terraform, Nvidia)
bin/                 shell wrappers for zil subcommands
architecture/        capability ownership maps, runtime evaluation structures
config/              test profiles
```

## Architecture: Layered with Two Runtimes

The project has **two independent implementations sharing a common semantic core**:

### Layer 1: Datalog Core (Zil/Datalog/)

The `Zil.Datalog` namespace implements a **pure clause-logic (Datalog) engine** inside Lean 4:

- **Types** (`Basic.lean`): `Value` (symbol/string/integer/boolean), `Atom` (object#relation@subject with optional attrs), `Pattern` (atom with variables), `Rule` (name + head + body literals with polarity), `Program` (facts + rules), `Substitution` (variable → value map)
- **Unification** (`Unify.lean`): term-level binding with occurs-free variables; attribute-map subset unification (all pattern keys must exist in ground atom)
- **Evaluation** (`Eval.lean`): `matchPattern` → `satisfyBody` → `satisfyNegativeBody` → `applyRule` → `deriveStep`. Fold-based fixpoint with `derive program fuel`. Stratified evaluation via `deriveStratified` for programs with negation.
- **Model theory** (`Model.lean`): `IsStratifiedModel` structure with fact-inclusion and closed-under-rules properties. Theorems proving correspondence between operational `applyRule` and declarative `RuleConsequence`. The key theorem `deriveStratified_is_model_of_fixed` proves that a stable bounded derivation IS a declarative stratified model.
- **Stratification** (`Basic.lean`): Dependency-graph analysis with Bellman-Ford-style relaxation (`computeStrata`). A negative dependency that participates in a cycle makes a program non-stratifiable — detected by `isStratified` via bounded reachability checks.
- **Lean environment integration** (`Environment.lean`): Uses `SimplePersistentEnvExtension` to store facts/rules across modules. Compiled `.olean` imports restore registered entries. Macro system (`zil_macro`/`zil_use`) for parameterized fact emission.

### Layer 2: Native ZIL Stack (Zil/ via Zil.Native)

A richer DSL layered over the engine with:

- **Theorem-shaped rules** (`Syntax/TheoremRule.lean`): `zil_theorem_rule` macro that looks like a Lean theorem statement but produces `Zil.Rule` values. Binders typed as `Zil.Node`, premises as relation expressions, conclusion as relation.
- **Syntax DSL** (`Syntax/Relation.lean`, `Syntax/Rule.lean`): `zil_fact node(A) ⟶[rel] node(B)`, `zil_rule`, `zil_typed_rule` with profiles.
- **Higher-level declarations** (`Core/Declaration.lean`): 20 declaration kinds (SERVICE, HOST, DATASOURCE, METRIC, POLICY, TM_ATOM, LTS_ATOM, PROOF_OBLIGATION, FORMALIZATION_TARGET, etc.), each with required-key validation, enum-domain checking, and deterministic lowering to canonical ZIL relations via `Declaration.lower`.
- **Trust levels** (`Core/Rule.lean`): `asserted` (registered), `graphDerived` (inferred), `certified` (backed by a Lean proposition and proof term).
- **Engine** (`Engine/Query.lean`, `Engine/Provenance.lean`): Stratified least-fixpoint with provenance tracking. Every fact in the closure records its origin (base or rule application with premise IDs and binding). `traceProgram` computes complete provenance. Query solving via `solve` with binding extension and negative-literal filtering.
- **Domain modules**: `Authorization.lean` (Zanzibar-style access decisions), `Formalization.lean` (scheduling formalization tasks by priority/dependency), `Impact.lean` (transitive change impact), `Workflow.lean` (action evidence with checkpoint/recovery/lease checks), `AgentContext.lean` (deterministic handoff bundles for AI agents), `ProofObligation.lean`, `TheoremAudit.lean`, `RecoveryAudit.lean`, `QueryGovernance.lean` (adaptive query plans with DSL profiles).

### Layer 3: Clojure Runtime (src/zil/)

- **Parser/Evaluator** (`core.clj`): Full `.zc` surface syntax parser — `object#relation@subject [attrs]`, `RULE ... IF ... THEN ...`, `QUERY ... FIND ... WHERE ...`, macros with `MACRO ... EMIT ...`
- **DataScript Backend** (`runtime/datascript.clj`): In-memory Datalog store using DataScript, giving the Clojure runtime query capabilities that mirror the Lean evaluator.
- **Adapters** (`runtime/adapters/`): REST, file, command, cucumber adapters for importing external data into the relational model.
- **Bridge layer** (`bridge/`): Interop with the Lean runtime — tuple-to-Lean compilation, workflow integration, proof token management, theorem CI, VStack verification chain.
- **Control plane** (`control/`): Capability-based access control, command routing, durable event store backed by SQLite.
- **Exchange protocol** (`Exchange/Protocol.lean`): JSON-based `ZIL-EXCHANGE/1` protocol for Lean worker ↔ Clojure control plane communication. Request/response with SHA-256 content addressing, capability checks, and deterministic per-operation arity.
- **Worker pool** (`worker/pool.clj`, `worker/client.clj`): Multi-worker process management with Lean native binaries.

## Key Architectural Insights

### 1. Stratified Negation Is a First-Class Concern

Most Datalog implementations treat negation as an afterthought or ban it outright. ZIL makes stratification a **compile-time gate**: every rule is checked for safety (all free variables in head and negative body must appear in positive body) and the program is checked for stratifiability (no negative dependency cycle). The `zil_rule` elaborator in `Environment.lean` rejects unsafe or non-stratifiable rules at definition time, not at query time.

The stratification algorithm (`computeStrata` in `Basic.lean`, `stratify` in `Engine/Query.lean`) is a Bellman-Ford-style relaxation: start each relation at stratum 0, then for each dependency edge, ensure `stratum(target) >= stratum(source) + (1 if negative else 0)`. Iterate to fixpoint. If the final stratum assignment isn't stable under one more relaxation pass, the program is rejected.

### 2. Two Semantics, One Proof of Correspondence

The Datalog model layer (`Model.lean`) proves that the **operational** fixpoint computation and the **declarative** model-theoretic semantics coincide. The theorem `ruleConsequence_iff_applyRule_mem` bridges the two views, and `deriveStratified_is_model_of_fixed` proves that a convergent stratified computation produces a declarative model. This is a rare example of a Datalog engine carrying its own correctness proof.

### 3. Provenance Tracking as a First-Class Feature

Rather than just returning query results, the Engine computes a full `Trace` — an array of `FactNode` values, each recording:
- Stable fact ID (deterministic, insertion-ordered)
- Stratum number
- Origin: `base` (registered fact) or `rule` (rule name, premise fact IDs, negative check results, variable binding)

This provenance data enables the CLI's `explain-query` and `explain-authorization` commands, which show exactly which facts and rules produced each inferred relationship. The `AgentContext` module uses this to build deterministic handoff bundles for AI agents — a list of relevant facts, their originating rules, and impact paths.

### 4. Dual-Representation Architecture

The project maintains **parallel but semantically equivalent** representations:
- `Zil/Datalog/` for the compact clause-logic core (list-based facts, simple atom structure)
- `Zil/` for the richer native stack (named nodes, theorem-shaped rules, profiles, declarations)

The Datalog layer is the ground truth for evaluation and model theory. The Native layer is the human-facing surface. `Zil.Datalog.Compat` bridges the two with aliases. This layering is unusual — most projects pick one representation and stick with it.

### 5. Declaration-to-Relation Lowering as Compiler Pass

The `Declaration.lower` function in `Core/Declaration.lean` is a mini-compiler: it takes one of 20 declaration kinds (SERVICE, HOST, TM_ATOM, LTS_ATOM, PROOF_OBLIGATION, etc.), validates its structure (required keys, enum domains, groundness), and compiles it into canonical ZIL relations. For TM_ATOM declarations, it even expands transition tables (map entries of `[fromState, readSymbol] -> [toState, writeSymbol, move]`) into individual `zil.transition` relations with structured attributes.

### 6. Agent Context as Compilation Target

`AgentContext.lean` is the most forward-looking module: given a set of changed nodes, it computes:
- All affected nodes via impact analysis
- All relevant provenance facts touching those nodes
- All originating rules
- All queries whose relation patterns touch the affected facts
- All formalization targets for the affected scope

This produces a deterministic `Bundle` structure — essentially an agent's context window, pre-filtered for relevance and with full provenance. It's designed so an AI coding assistant can receive exactly the facts, rules, and targets relevant to a specific change, rather than needing the full knowledge base.

### 7. Causal Revision Tracking

`Core/Revision.lean` defines a causal event model with `before(e1, e2)` relations that must be irreflexive, transitive, and acyclic. The `Codec/Revision.lean` provides `ZILR/1` revision log format with `ZILX/1` snapshots and `ZILD/1` deltas. The `causal-check` CLI command validates the causal graph in a revision envelope. This is a lightweight alternative to full CRDT-based revision control, optimized for the relational fact domain.

## Design Trade-offs

- **Simplicity over performance**: The evaluation engine uses flat list representations (`List Atom`, `List Rule`) rather than indexed structures. For the scale of project-level relationship tracking (hundreds to low thousands of facts), this is sufficient. The Datalog `Index.lean` module exists but the core evaluator doesn't use indexing.
- **Determinism over parallelism**: Fact order within a stratum is preserved, and fact IDs are insertion-ordered. This makes provenance deterministic but precludes parallel rule evaluation within a stratum.
- **Compile-time rejection over runtime error handling**: Safety violations (unsafe rules, non-stratifiable programs, invalid declarations) are caught at elaboration time (`command_elab` in the Lean environment) rather than at query time. This is the right choice for a tool that integrates with Lean's type-checking pipeline.
- **Lean-native DSL over standalone language**: The theorem-shaped rule syntax (`zil_theorem_rule`) integrates ZIL rules into Lean's module system, making them persist across imports via environment extensions. This means ZIL rules participate in Lean's compilation model rather than existing as a separate preprocessing step.
- **Dual runtime cost**: Maintaining both the Lean native implementation and the Clojure/DataScript runtime means spec changes must be implemented twice. The exchange protocol and conformance tests (`zil-conformance`) bridge this gap by ensuring both runtimes produce equivalent results.

## Comparison to Related Systems

### vs. Google Zanzibar (as described in the paper)
Zanzibar focuses exclusively on authorization decisions with a specific consistency model (global, with Zookies for staleness). ZIL borrows the tuple shape (`object#relation@subject`) but applies it to general project relationships. Zanzibar uses userset rewriting for group membership; ZIL uses Horn rules for all derivation, including authorization.

### vs. Soufflé Datalog
Soufflé is a high-performance Datalog compiler with semi-naive evaluation, magic-set optimization, and C++ code generation. ZIL is a pragmatic, embedded Datalog that prioritizes integration with Lean's type system over raw performance. Soufflé is designed for program analysis at scale; ZIL is designed for project metadata at human scale.

### vs. Datomic/DataScript
Datomic and DataScript use Datalog as a query language over EAVT (entity-attribute-value-time) stores with explicit time dimension. ZIL uses Datalog as both a query language and a fixpoint-based derivation engine. ZIL's Clojure runtime uses DataScript as its storage backend, so the two are complementary — DataScript provides the indexed store, ZIL provides the rule evaluation and provenance layer.

### vs. TLA+ and Alloy
Both are specification languages with model checking. ZIL is a runtime — it evaluates rules to closure and answers queries against that closure. It doesn't explore the state space of concurrent systems. However, its `tmAtom` and `ltsAtom` declaration kinds (Turing machines and labeled transition systems) treat state machines as first-class data, blurring the line between specification and runtime.

## Key Source Files

| File | Lines | Role |
|------|-------|------|
| Zil/Datalog/Basic.lean | 203 | Core types: Atom, Rule, Program, stratification |
| Zil/Datalog/Eval.lean | 65 | Horn-rule evaluation engine |
| Zil/Datalog/Model.lean | 104 | Declarative model theory and correctness proofs |
| Zil/Datalog/Unify.lean | 54 | Pattern-atom unification with attributes |
| Zil/Datalog/Environment.lean | 345 | Lean environment extensions, syntax, command elaboration |
| Zil/Core/Declaration.lean | 435 | Declaration kinds, validation, lowering to relations |
| Zil/Core/Program.lean | 52 | Native program representation |
| Zil/Engine/Query.lean | 235 | Stratified fixpoint, query solving, entailment |
| Zil/Engine/Provenance.lean | 338 | Fact provenance, traces, explanations, rendering |
| Zil/CLI/Main.lean | 317 | Native CLI with ~20 subcommands |
| Zil/Workflow.lean | ~120 | Action evidence with checkpoint/recovery checks |
| Zil/AgentContext.lean | 262 | Agent context bundle construction |
| Zil/Exchange/Protocol.lean | 216 | JSON protocol for Lean/Clojure interop |
| src/zil/core.clj | ~600 | Clojure parser/evaluator |
| spec/zil-formal-core-v0.1.md | 427 | Formal semantics specification |

## Dependencies

- **Lean 4**: v4.31.0 (pinned in lean-toolchain)
- **Clojure**: 1.11.1
- **DataScript**: 1.7.8
- **SQLite JDBC**: 3.48.0.0
- **Clojure data.json**: 2.5.1
- **Lake**: Lean 4 build system (lakefile.lean)
- **Docker**: Test infrastructure (compose.test.yml, Dockerfile.test)
