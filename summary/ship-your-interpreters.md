---
url: https://www.basis.ai/blog/ship-your-interpreters/
title: "Ship Your Interpreter"
author: Kiran Gopinathan, Nikhil Pimpalkhare, Yiyun Liu, Zenna Tavares
date_fetched: 2026-10-24
date_published: 2026 (undated on page)
topics:
  - guardrails-and-feedback-loops
  - ai-research-and-models
---

Basis reports an experiment in machine-verified software: LLM agents wrote a formal semantics for an extended WHILE language in Lean 4, then proved that the language's real C interpreter — compiled to RISC-V and checked against a formal machine model — *refines* that semantics, i.e. agrees with it on what every program prints. The interpreter is treated as ground truth, replacing testing entirely.

The motivating bottleneck is specification: as LLMs get better at theorem proving, the hard part is knowing whether the thing you proved is the thing you care about. Testing can't validate a semantics — their example is a 59-character function name that tripped a 64-byte buffer in `stringify`, invisible to every test but fatal to any proof built on top. The proof process itself caught four semantic errors (64-bit wrapping arithmetic, `for`-initialiser status handling, differing `print` vs concatenation rendering, the 63-byte name cut) plus errors in the LLM-written C library lemmas; every error surfaced as an obligation that would not close.

The hard part was memory reasoning. A fixed-footprint Hoare-triple spec for `malloc` is provably impossible — the LLMs proved no fixed address set works — so they ported MachCSL-style separation logic to the new iris-lean, making the footprint heap-dependent. The proof peaked at ~1M lines of Lean (two-thirds script-generated, one rule per machine instruction across ~11,800 instructions) and was later refactored to under 200K. On the trusted semantics they cheaply built four verified tools: a type checker, a program logic, an abstract interpreter, and a verified WHILE-to-RISC-V compiler (2.7–8.2× fewer retired instructions than the interpreter). Next targets: Lua 5.4.7 and OCaml via its self-hosting bytecode; north star CPython.
