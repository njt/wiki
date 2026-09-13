---
url: https://projectzero.google/2026/09/maccconc-race-condition.html
title: "MAccConc: Exploring Linux Kernel Race Condition Interleavings"
author: Google Project Zero
date_fetched: 2026-09-13
date_published: 2026-09
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

A Google Project Zero researcher released **MAccConc** ("Memory Access Concurrency"), tooling for exploring the possible interleavings of multi-threaded test cases against the Linux kernel. The motivating pain is that security bugs are often race conditions: you can neither confirm a suspected bug nor write a regression test after a fix, because the failure needs one specific thread interleaving that random execution rarely produces. The old artisanal method — recompiling the kernel with hand-placed `mdelay()` calls, or DTrace `chill()` probes — is slow and trial-and-error.

The toolset has three parts: an automatic tester that exhausts A-B-A interleavings of a two-thread test case, a terminal UI for manual exploration, and a GUI that shows per-thread call graphs with memory accesses annotated (read/write/free, address, size, prior value) and lets you right-click two accesses to create an ordering constraint between them.

The machinery underneath: memory accesses are collected in-kernel by reusing KCOV (fed by outline-mode ASAN instrumentation, one helper call per access), pairs of cross-thread accesses that could interact are identified as "communication points" (the idea is taken from the SKI paper), and each access is given a stable cross-run identifier via **count-augmented stack traces** — each element is a function plus an occurrence count, e.g. "the second call to `_raw_spin_unlock` inside `unix_stream_read_generic`". This avoids SKI's VM-snapshot approach; it required an LLVM SanitizerCoverage feature (function entry/exit events) that landed in LLVM 23.1.0. A new `KCOV_SET_DI` ioctl then performs delay injection: wake/wait actions on a shared flag array let userspace force specific orderings, either constraint-style (A happens before B) or fully specified (context-switch-style, used by the A-B-A tester).

Status: kernel patches were posted for upstream review alongside the post (a branch is on GitHub; LLVM support has landed); the CLI handles two threads, the GUI more. Future work includes TSAN-style lock-event feedback, faster detection of impossible orderings, Snowboard-style fuzzing on top of the instrumentation, and a DWARF 6 `DW_AT_alloc_type` proposal for typing memory-access traces.
