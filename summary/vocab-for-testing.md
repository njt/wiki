---
url: https://alperenkeles.com/posts/vocab-for-testing/
title: "A Contemporary Vocabulary for Testing"
author: Alperen Keleş
date_fetched: 2026-11-26
date_published: 2026 (undated on page)
topics:
  - software-engineering-craft
  - guardrails-and-feedback-loops
---

Keleş argues that the standard testing vocabulary — unit/integration/E2E, stress/load/penetration — is undefined and "based on vibes," and proposes a three-axis decomposition instead: the **scope** of the test unit (function/class/module/system, stateful or stateless), the **purpose/hypothesis** the test falsifies (correctness, performance, security, resilience), and the **oracle** that decides failure. He then separates oracles from **methods** of producing inputs, which lets him reframe everything from tinker testing and snapshot testing to type systems, fuzzing, and deterministic simulation testing as points in one design space — with an open bias toward random testing, his own specialty.

On scope: functions consume arbitrary language-structured inputs while systems consume serialized, pre-validated interactions, so system testing gets a natural input discriminator that function signatures lack. Stateful tests can be made stateless by lifting state into the input-output pair, though complex realistic states are often impossible to construct — in which case you test the system as a simulation over its public API. Design by Contract, refinement types, and better domain modelling shrink the over-relaxed input spaces of programming languages.

On oracles: humans (his "tinker testing" — which traditional tests merely codify), concrete examples (including expect/golden/snapshot testing), inline assertions (DbC pre/postconditions, Tiger Style's aggressive production assertions), and properties — global invariants, metamorphic relations, roundtrip, compatibility/differential testing (GCC, SQLite, S3, Cedar), idempotency. Properties are pure oracles detached from method; LTL/TLA+ extend them into temporal logic.

On methods: static analysis (type systems themselves are tests; refinement-type checkers like Liquid Haskell, Flux, Dafny check properties statically), enumeration, and randomness — generation, mutation, coverage-guided fuzzing (AFL, OSS-Fuzz, CSmith, SQLancer), property-based testing libraries (QuickCheck, Hypothesis, proptest), model-based simulation for stateful systems (Turso, SQLancer), and fault injection / Deterministic Simulation Testing (FoundationDB, TigerBeetle, Antithesis).
