---
url: https://bend-lang.com/
title: "Bend — a fast language that blocks AI mistakes via proof"
author: "HigherOrderCO (Victor Taelin); author not stated on page"
date_fetched: 2026-09-19
date_published: "unknown (not stated on page)"
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

Bend (bend-lang.com) is a new programming language pitched at a post-AGI world where humans "will eventually stop writing and reading code": a **fast language that blocks AI mistakes via proof**, offering C speed, CUDA-style parallelism, Lean-style proofs, and Python-shaped syntax. The page is the product's landing pitch plus its full language guide (GUIDE.md), and the pitch is aimed squarely at coding agents rather than human developers.

The agent workflow is the product. You add four lines to your AGENTS.md — run `bend guide` to learn it, keep important rules in `LAWS.bend`, run `bend PROOF.bend` before committing, parallelize whenever possible. `LAWS.bend` is written by the human and states open claims (e.g. `law you_cant_win: for moves: List<Move> ... is_won(board) == False{}`); `PROOF.bend` is written by the AI and proves each law with a def of the same name. The proof checker is the commit gate: it fails while any law is open or false. The site's key engineering claim is checker speed — Lean and Rocq can take minutes on a mid-sized codebase, "Bend takes a second at most, so an AI agent can check after every change." The demo: asked to "make the board wrap around", the agent shipped the bug without laws; with laws, it "had to retry until it built a wall and proved the law holds. Merging a bug is mathematically impossible: it is a *theorem*." The compression the whole project rests on: "`LAWS.bend` is `AGENTS.md` backed by proof."

The language design serves that checker. Bend is affine by default (a variable is used at most once; quantities &0/&1/&2 mark erased/affine/reusable), does "almost no inference" — verbose annotations are the price of a fast checker and precise errors — mandates termination, bans mutual recursion (a function that never returns "could prove anything"), and has no tactics: a proposition is a type and a proof is a def of that type, with `{==}` for reflexivity, `%e` rewrites, and recursive calls as induction hypotheses. Parallelism falls out of purity: `a b = f(x) g(y)` is a parallel call spread over every core by a contention-free binary fork-join scheduler, `!` marks a GPU call, and the compiler emits a single C file that serves as both CPU program and GPU kernel. IO is a Haskell-style monad over a Node.js-style event loop; arrays give in-place mutation without giving up purity, via single ownership.

The docs are candid about the edges: Bend is young ("expect bugs, and please report them"), `@unsafe` defs escape the proof guarantees, the type theory admits `Type : Type` and no positivity check — kept consistent by a wall between "live" code (must terminate) and "dead" code (types, erased arguments, equations) — and the Lean mechanization (`bend.lean`) lags the implementation. The guide's closing pointer is telling: `bend guide shaders` prints a shader tutorial "written by AIs for AIs." The language's intended reader is the model.
