---
url: https://github.com/happi/theBeamBook
title: "The BEAM Book: Understanding the Erlang Runtime System"
author: Erik Stenman (happi) and contributors
date_fetched: 2026-11-13
date_published: 2025 (first edition print; repo ongoing, second edition in progress)
topics:
  - developer-tools
  - distributed-systems
---

The BEAM Book is Erik Stenman's open-source (CC BY 4.0) deep-dive into the Erlang RunTime System (ERTS) — the ~15,000-line AsciiDoc manuscript behind the printed first edition (2025, ISBN 978-91-531-4253-9), with runnable Erlang/C example code per chapter and a second edition in progress.

Part I ("Understanding ERTS") walks the runtime stack chapter by chapter: the compiler and its intermediate formats, processes, the type system, the BEAM virtual machine itself (a register machine, not a stack machine), the .beam file format, the loader, instruction set, scheduling, the memory subsystem, garbage collection, I/O and ports, distribution, and the C/NIF interface. Part II ("Running ERTS") covers operations: shell tooling, debugging, tracing, testing and profiling.

The framing thesis of the introduction: writing a bytecode interpreter is "just the tip of the iceberg" — what makes an industrial runtime is everything underneath (schedulers, per-process heaps, allocators, distribution, signals). The book treats the node — a named, clusterable runtime instance — as the unit of deployment, and reductions (a fuzzy per-function-call work unit tested against a quota at each function call) as the currency of its preemptive scheduler. Distributed Erlang is built into the runtime: short-name and long-name nodes form a fully connected cluster, with the caveat that the two naming schemes cannot interoperate.
