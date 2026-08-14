---
url: https://github.com/boldsoftware/meat
title: Meat — Reading Diff
author: boldsoftware
date_fetched: 2026-08-14
---

Meat is a Go CLI that abridges a code diff into a **reading diff**: the same change, rewritten to keep only what a senior reviewer actually needs to read. The premise is that reviewing agent-written code no longer means sweating style, nil-checks, or imports — those are covered by the compiler and tests. What remains is the *change to the program*: what moved, where data came from, what new behavior appeared. Meat strips the mechanical noise and shows you the meat.

The architecture's central trust move is that **the model never authors output text**. Meat sends the model a numbered unified diff and asks for an edit plan — `remove` ranges, `fold` ranges, and single-line `replace` elisions, all in original line coordinates. A deterministic compiler then validates the plan (elision projections, move symmetry, Python structure rules, retained metadata) and mechanically renders the reading diff from the immutable input. The output is always a compressed subsequence of the original, so the model cannot invent code, comments, or behavior.

Around that core sit several subsystems: a per-language **mandatory import pass** that removes imports/includes/requires/use declarations automatically (Go, Python, JS/TS, Rust, C/C++, Java); **exact move detection** that pairs relocated blocks across hunks/files and enforces symmetric treatment so a move reads as relocation, not one-sided deletion; **chunking** that splits oversized diffs (up to 4 MB) at file→hunk→sub-hunk boundaries and merges per-chunk results; and a **cache** keyed by the SHA-256 of (rubric hash + model + diff) so unchanged re-runs are instant.

It is provider-agnostic with zero third-party dependencies: built-in backends speak the OpenAI Responses API (default `gpt-5.6-sol`) and Anthropic Messages API (default `claude-opus-4-8`) over the Go standard library. Install with `go install meat.dev/cmd/meat@latest`, then run `meat` to review the latest commit, or pipe any diff in.
