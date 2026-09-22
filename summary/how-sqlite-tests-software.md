---
url: https://gist.github.com/b7d09e2d36ba4636bd48091e7fe9d13e
title: "How SQLite Tests Software"
author: D. Richard Hipp
date_fetched: 2026-09-13
date_published: 2026
topics:
  - software-engineering-craft
---

A 2026 conference talk in which D. Richard Hipp — creator of SQLite and one of its three committers — explains how the most widely deployed database in the world became that reliable. The origin story is a broken dependency: a flaky Informix server produced user-facing failures Hipp couldn't control, so he eliminated the server and wrote a database as a single file.

The talk's core is the testing discipline. SQLite holds 100% MCDC (modified condition/decision coverage): every machine-code branch is exercised both ways, and every bit in a bitmask test must independently affect the outcome. It is measured with GCC's gcov against the *deliverable* object code, not a debug build, via a custom C harness called TH3 — over six times larger than the SQLite source it tests. Reaching 100% MCDC in 2009, Hipp says, stopped the flood of external bug reports almost overnight.

Reliability is designed in, not bolted on. The product was re-architected for testability: a pluggable VFS and a published `sqlite3_test_control` API (shipped in every production build) let tests inject deterministic I/O errors, simulate power-loss crashes, rig the memory allocator to fail on the Nth allocation, and trip the `faultsim` hook to test error-recovery paths. About 15–20% of SQLite's source code exists only for testing, and Hipp treats that as a feature, not waste.

Beyond coverage, the talk covers comments and asserts as executable, testable artifacts; fuzzing and semantic fuzzing for what MCDC misses; and AI as a bug-finder (an AI found a pathological quicksort stack overflow) whose fixes Hipp distrusts. His closing claim: all this is necessary but not sufficient — SQLite's adoption also required "an element of providence," forces outside anyone's control.
