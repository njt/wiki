---
url: https://wiki.alcidesfonseca.com/blog/full-stack-verification-is-the-new-trend/
title: "Full-Stack Verification Is the New Trend"
author: Alcides Fonseca
date_fetched: 2026-11-06
date_published: 2026-11
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

A survey of a recent wave of announcements — Harmonic's Aristotle, SF-Tensor's formalization of NVIDIA PTX in Lean, &Truth (with Clark Barrett, Ilya Sergey and Sebastian Ullrich), Theorem./Anthropic's RISC-V verification, AutoResearch's WASM-to-syscall proofs — all converging on *full-stack* formalization: proving correctness across whole software and hardware stacks rather than a small verified core or a simplified model.

Fonseca then takes a position. For years his sweet spot was liquid types: restricted value-dependent types discharged automatically by SMT solvers, no proofs to write. That trade-off mattered because almost nobody had the expertise to fully verify a large system (seL4 being the exception). Frontier models' "superhuman ability to generate proofs" has eroded that moat, and subsidized token prices make full-stack verification experiments cheap right now — though he expects the image to flip back if token prices rise to real cost.

If you go full-stack, you need a verification-first language, and he argues Lean is the ideal candidate (Dafny and his own aeon as alternatives; Rocq's OCaml extraction "not such a significant advantage"). But Lean must improve: faster type checking and a much better optimizer — the part Bend got right, since "the next mainstream language will require both verification and speed." He closes with a prediction that people will stop reading source code at all and rely on new techniques for confidence in generated code.
