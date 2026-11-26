# Unit and Integration Tests (matklad)

matklad (Alex Kladov, rust-analyzer) argues the unit/integration test dichotomy is confused vocabulary and replaces it with two axes — **purity** (roughly: how impure, hence slow and flaky, the test's runtime is) and **extent** (how much of the codebase the test exercises) — with the prescription to optimize purity ruthlessly and leave extent alone.

---

## What it argues

The unit/integration debate is unresolvable because the terms are overloaded: "unit testing" is used as a synonym for automated testing generally, and Cargo repurposed the words for compilation-model properties entirely orthogonal to the traditional fuzzy meaning. The replacement model is lifted from the SWE book (Software Engineering at Google), with two deliberate deviations.

**Purity** is the axis that actually matters for engineering outcomes. Test speed is *categorical* — an order-of-magnitude ladder, not a dial:

- Single-threaded pure computation
- Multi-threaded parallel computation
- Multi-threaded concurrent computation with time-based synchronization and disk access
- Multi-process computation
- Distributed computation

Each rung costs roughly half an order of magnitude of runtime, and buys more flakiness and fragility to unrelated environment changes. The functional-programming concept of purity generalizes it well: pure tests do little-to-no IO, depend on no timings, and have discrete notches of impurity (threads, time, filesystem, network, processes) rather than a continuum.

**Extent** is how much code the test exercises, directly or indirectly. The key move is decoupling it from speed: "running more code doesn't mean that the code will run slower. An infinite loop takes very little code." What matters for performance is not whether tests touch persistence but whether persistence in tests is an in-memory hash-map or an out-of-process server. Extent is diagnostic of architecture but not a target: mocking *your own code* (as opposed to mocking impure IO) reduces fidelity and makes refactors brittle. One exception: compilation time in layered systems, where driving layer A through layer C forces rebuilding B and C.

The worked example is the clincher: a rust-analyzer test for trait-method completion is trivially 100% pure (single thread, no IO) yet of maximal extent (lexer, parser, name resolution, type checking all must handle incomplete code). Unit or integration? The question is meaningless; purity/extent answers instantly.

Versus the SWE book: they say "size" where he says "purity," and their second axis is "scope" — how large the bit actually being tested is. He finds scope useful prescriptively but too fuzzy to classify with; the same test's scope stumps him ("it tests not so much the completion logic itself, as all the underlying infrastructure for semantic analysis").

## Key quotes

> "Tests longer than attention span obliterate productivity."

The claim that edit-to-result latency correlates super-linearly with nearly everything else worth having (performance, defect fix time, security) is doing a lot of work here — purity isn't an aesthetic preference, it's the input to the whole feedback loop.

> "Artificially limiting the extent of tests by mocking your own code (as opposed to mocking impure IO) reduces fidelity of the tests, and makes the code more brittle in the face of refactors."

The sharpest line in the piece, and the one most contrary to prevailing dogma. The mock-everything school treats extent as the enemy; matklad says mock the *world*, not your *design* — the coupling your tests reveal is real information about your architecture.

> "The test is very easy to classify in the purity/extent framework. It's 100% pure — no IO, just a single thread. It has maximal extent."

## Key themes

#concept #pattern — a two-axis vocabulary replacing a false dichotomy.

## Opinionated take

This is one of the best short essays on testing ever written, and it has aged better than almost anything else in the genre. The insight that test speed is categorical rather than numerical is genuinely load-bearing: teams that treat "fast tests" as a gradient never notice they've crossed a rung into multi-process flake territory until the suite is already untrustworthy. And the purity/extent split dissolves, rather than resolves, the eternal mockist/classicist war — both sides were arguing about extent when the actual battle is about purity.

The arguable weakness: extent's "let it be natural" prescription is quietly doing more work than admitted. Compilation-time coupling is dismissed as "one potential exception," but in many real codebases (anything with heavy codegen or slow type checkers) it's the dominant constraint, and extent discipline is the only lever. That said, matklad ships rust-analyzer, a codebase famous for fast incremental builds — his environment genuinely lets extent be free. Readers in slower ecosystems should apply the caveat more forcefully.

## Why it matters here

For the wiki's agentic-testing cluster, this essay supplies the vocabulary that several newer sources reach for without naming. It strengthens [[A New Era for Software Testing]]: antirez's argument that agents excel at impure, integration-flavored QA work presupposes exactly the purity ladder matklad drew — agents make the expensive rungs cheap to run, which changes the purity calculus but not the taxonomy. It nuances [[Agentic Verification]]: Osmani's loops (E2E as ground truth, property tests as constraints) can be read as a purity/extent portfolio, and matklad's warning about mocking your own code maps directly onto the failure mode of verifying agent output against an agent-written test double. And it complicates [[TDD Inside the Agent Loop — Theater or Actual Value?]]: the Thoughtworks finding that prescribed TDD adds no quality at 3–8x tokens may partly reflect that TDD ritual optimizes extent-style structure rather than purity — the one axis matklad says isn't worth optimizing.

---
*Sources: [[raw/unit-and-integration-tests-html]], [[summary/unit-and-integration-tests-html]]*
*Last updated: 2026-11-26*
