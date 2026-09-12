---
url: https://github.com/jagg-ix/zil-lean
title: "ZIL Lean"
author: jagg-ix
date_fetched: 2026-08-01
date_published: 2026
topics:
  - agent-architecture
---

ZIL (ZIL is a Language) is a small relational language for describing named objects, their relationships, and Horn-clause rules that derive additional relationships. It uses relation tuples (`subject ── relation ──▶ object`) influenced by Google's Zanzibar authorization paper, applied to project metadata: requirements coverage, dependency tracking, change impact analysis, formal verification scheduling, and agent context handoff.

The project has two independent implementations sharing a common semantic core. The Lean 4 side embeds a Datalog engine with stratified negation, model-theoretic correctness proofs, and a theorem-shaped rule DSL that integrates with Lean's type-checking pipeline. The Clojure side provides a `.zc` file parser, a DataScript-backed runtime, adapters for importing external data, and a bridge layer for interop with the Lean runtime via a JSON exchange protocol.

Key architectural choices: stratification is a compile-time gate — unsafe or non-stratifiable rules are rejected at elaboration time. The engine computes full provenance traces for every derived fact, recording origin rules, premise bindings, and stratum numbers. A `Declaration` layer lowers 20 higher-level declaration kinds (SERVICE, HOST, PROOF_OBLIGATION, etc.) into canonical relations through a compiler-like pass. The `AgentContext` module builds deterministic handoff bundles for AI agents — pre-filtered facts, rules, and formalization targets relevant to a specific change.

Design trade-offs favor simplicity and determinism over performance: flat list representations rather than indexed structures, insertion-ordered fact IDs rather than parallel evaluation, and dual-runtime maintenance cost offset by conformance testing.
