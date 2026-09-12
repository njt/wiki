# ZIL — Relational Language Embedded in Lean 4

ZIL is a domain-specific relational language for tracking project metadata — requirements, dependencies, proofs, tasks, and change impact — as directed subject→relation→object tuples. Implemented as a dual system with a Lean 4 formal DSL and a Clojure/DataScript runtime, it applies Zanzibar-style relation tuples to the wider problem of making project relationships machine-queryable. Its innovation is embedding Datalog-style Horn rules with stratified negation directly inside a proof assistant, with full provenance tracking and correctness proofs linking operational evaluation to declarative model theory.

#project #tool #formal-verification #datalog #lean-prover

---

## Architecture

ZIL has **three layers** built on two runtimes that share a common semantic core:

**Layer 1: Datalog Core** (`Zil/Datalog/`). A pure clause-logic engine in Lean 4: atoms (`object#relation@subject` with optional key-value attributes), patterns with variables, rules with positive and negative body literals, and a bottom-up fixpoint evaluator. The evaluator (`Eval.lean`) performs naïve evaluation (fold-based, no semi-naïve optimization), with stratified evaluation for programs containing negation. The model layer (`Model.lean`) proves the correspondence between operational rule application and declarative model theory — `deriveStratified_is_model_of_fixed` establishes that a convergent stratified computation IS a declarative model.

**Layer 2: Native ZIL Stack** (`Zil/` via `Zil.Native`). A richer DSL layered over the engine. Key modules:
- **Theorem-shaped rules** (`Syntax/TheoremRule.lean`): `zil_theorem_rule` macro produces `Zil.Rule` values from Lean theorem-shaped syntax
- **Higher-level declarations** (`Core/Declaration.lean`): 20 declaration kinds (SERVICE, HOST, TM_ATOM, LTS_ATOM, PROOF_OBLIGATION, FORMALIZATION_TARGET, etc.) with validation and deterministic lowering to canonical relations
- **Trust levels** (`Core/Rule.lean`): `asserted` → `graphDerived` → `certified` (backed by a Lean proposition and proof term)
- **Provenance tracking** (`Engine/Provenance.lean`): Every fact in the closure records its origin (base fact or rule application with premise IDs and binding), enabling `explain-query` and `explain-authorization`
- **Domain modules**: Authorization (Zanzibar-style access decisions), Formalization (priority-based task scheduling), Impact (transitive change analysis), AgentContext (deterministic AI-agent handoff bundles), Workflow (action evidence with checkpoint/lease/recovery checks)

**Layer 3: Clojure Runtime** (`src/zil/`). A DataScript-backed parser/evaluator for the `.zc` file format, with REST/file/command adapters, a plugin system, and a bridge layer for Lean interop. The `ZIL-EXCHANGE/1` JSON protocol connects a Clojure control plane to Lean worker processes.

**Compilation flow**: `.zc` source → `Parser/DeclarationProgram.lean` parses into `Zil.Program` → declarations validated and lowered to canonical facts → rules checked for safety and stratifiability → stratified least-fixpoint evaluation → queries solved against the closure.

## Key Techniques

**Stratified negation as a compile-time gate.** Most Datalogs either ban negation or handle it at query time. ZIL performs Bellman-Ford-style stratum computation (`computeStrata` in `Basic.lean`, `stratify` in `Engine/Query.lean`) and rejects non-stratifiable programs at elaboration time — the `zil_rule` command elaborator checks both safety (free variables bound in positive body) and stratifiability (no negative dependency cycle).

**Declaration-to-relation lowering as a compiler pass.** `Declaration.lower` (`Core/Declaration.lean:417`) takes 20 declaration kinds, validates structure (required keys, enum domains, TM/LTS consistency), and compiles each to canonical ZIL relations. TM_ATOM declarations expand transition tables — `[fromState, readSymbol] → [toState, writeSymbol, move]` entries — into individual `zil.transition` facts with structured attributes. Service declarations automatically emit inverse dependency facts (`uses` → `usedBy`).

**Provenance tracking for explainability.** The Engine's `traceProgram` computes a complete provenance closure — every fact gets a stable ID, stratum number, and origin (base or rule application with premise IDs and variable binding). This enables step-by-step explanations for any inferred relationship. The `AgentContext` module uses this provenance to build deterministic handoff bundles for AI coding agents — pre-filtered facts, rules, queries, and targets for a specific change scope.

**Lean environment extension for persistence.** Facts and rules registered via `zil_fact`/`zil_rule` are stored in `SimplePersistentEnvExtension` entries. Compiled `.olean` files restore these entries on import, collapsing repeated diamond imports into one relation. This means ZIL facts participate in Lean's compilation model — they survive across module boundaries and rebuilds.

**Operational/declarative correspondence proof.** `Model.lean` proves `ruleConsequence_iff_applyRule_mem` and `deriveStratified_is_model_of_fixed` — the operational fixpoint computation and the declarative model theory coincide. For frozen programs, `native_decide` can discharge the stability premise, making the proof fully automated.

## Design Decisions

**Simplicity over performance.** The evaluator uses flat `List` representations rather than indexed structures. For project-scale metadata (hundreds to low thousands of facts), the O(n²) matching cost is negligible. The `Index.lean` module exists but the core evaluator doesn't use it.

**Determinism over parallelism.** Facts are insertion-ordered, IDs are deterministic, and provenance records the first derivation witness. Within a stratum, rules are evaluated sequentially. This makes provenance reproducible but precludes in-stratum parallel evaluation.

**Compile-time rejection over runtime errors.** Unsafe rules and non-stratifiable programs are rejected during `command_elab` — the same phase that type-checks Lean code. This integration with the compiler pipeline means ZIL errors surface the same way as type errors.

**Dual runtime with conformance testing.** Maintaining both Lean and Clojure implementations is expensive but provides cross-validation. The `zil-conformance` tool renders canonical semantic reports from both runtimes and diffs them.

**Lean-native DSL over standalone language.** By embedding the syntax as Lean macros rather than a standalone parser, ZIL facts and rules live in `.lean` files alongside regular code, participate in the module system, and benefit from Lean's IDE support (go-to-definition, error reporting).

## Comparison Notes

- **vs. Google Zanzibar**: Zanzibar is a production authorization system with global consistency and Zookies. ZIL borrows the tuple shape (`object#relation@subject`) but applies it to general project relationships. Zanzibar uses userset rewriting for group membership; ZIL uses Horn rules for all derivation, making it a superset of Zanzibar's expressive power.

- **vs. Soufflé Datalog**: Soufflé compiles Datalog to optimized C++ with semi-naive evaluation and magic-set optimization. ZIL is an embedded DSL that trades performance for integration with a proof assistant and full provenance tracking. Soufflé targets program analysis at scale; ZIL targets project metadata at human scale.

- **vs. [[We Have Proof Automation Now]]** (Adam Langley's Lean+LLM proof generation): ZIL and Langley's approach are complementary. ZIL tracks *what* has been proven and *how* declarations relate; Langley's approach generates *the proofs themselves* using LLMs. ZIL's formalization scheduling (`Formalization.lean`) could direct proof-generation agents at the highest-priority unverified targets.

- **vs. [[The Coming Need for Formal Specification]]**: ZIL is a concrete implementation of the kind of specification infrastructure that essay argues for — machine-readable relational metadata that connects specifications to implementations, with provenance and trust levels. The `certified` trust level is the bridge: a ZIL rule backed by a Lean proof term means the relationship has been mechanically verified, not just asserted.

- **vs. typical agent memory systems**: Most agent memory systems use vector embeddings or knowledge graphs for retrieval. ZIL takes a different approach: it pre-computes relevant context for a given change via transitive impact analysis and provenance filtering (`AgentContext.lean`). Rather than the agent querying a knowledge base, the system computes a deterministic bundle of exactly the facts, rules, and targets relevant to the task.

---
*Sources: [[raw/zil-lean]]*
*Last updated: 2026-08-01*
