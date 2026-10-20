---
url: https://gavinray97.github.io/blog/design-by-contract-and-effects-for-llms
title: "Design by Contract and effects are essential for LLM-generated code"
author: Gavin Ray
date_fetched: 2026-10-20
date_published: 2026 (undated)
topics:
  - software-engineering-craft
  - guardrails-and-feedback-loops
---

Gavin Ray argues that two programming-language features — Design-by-Contract (preconditions, postconditions, invariants written into signatures) and explicit effects (capabilities like `fs.read`, `alloc`, `state.write` declared in a method's type) — become roughly "tenfold" more valuable when most code is authored by LLMs rather than humans.

The core claim is about the development loop: if you prompt an LLM for an implementation and verify with tests, what you want is "code that verifiably does what it says on the tin." Contracts make intent machine-checkable — e.g. `change_email` must `ensures user.email_verified == false`, catching the classic bug where a generated implementation updates the address but leaves the verified flag set. Invariants suit data structures especially well (MinHeap ordering, B-tree node shape).

Effects make operational behaviour visible at the call site: `load_settings` that touches the filesystem and one that parses an in-memory string return the same type but have meaningfully different behaviour, and an effect-aware signature says so. Ray's headline payoff is compiler-generated reports of *semantic* changes in a PR — "Effects added: PaymentProcessor.process + net.connect + retry.nondeterministic; Postcondition weakened: Ledger.append" — which is exactly the level at which human reviewers of generated code actually want to read.

The piece is a short, code-first argument (pseudo-syntax examples throughout) with a disclaimer that no prose was machine-generated. It stops short of naming concrete languages or showing a working toolchain; it is a thesis about where language design should go, not a report of a system that exists.
