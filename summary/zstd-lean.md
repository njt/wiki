---
url: https://www.imperialviolet.org/2026/07/26/zstd-lean.html
title: "We have proof automation now"
author: Adam Langley
date_fetched: 2026-07-29
date_published: 2026-07-26
topics:
  - software-engineering-craft
---

Langley explores whether LLMs can automate the proof-writing burden that has
historically made dependently-typed languages prohibitively expensive. He builds
a Zstandard decompressor in Lean as a testbed and reports that modern LLMs can
produce non-trivial correctness proofs — type-checked, with no `sorry`s — in
about 20 minutes for a fraction of a $20/month subscription.

The post covers three intertwined threads. First, the history and appeal of
dependent types: languages like Coq/Rocq and Lean let you encode arbitrarily
subtle invariants in the type system, but the tradeoff has been enormous proof
effort (the seL4 retrospective found ~10× more time spent proving than
implementing). Second, a lucid explanation of Zstandard's FSE entropy encoder:
a finite-state-machine approach that assigns symbols fractional-bit costs on
average, and forces the decoder to process blocks backwards. Third, the
practical experience of writing in Lean — its strict evaluation, monadic
syntactic sugar, and the sharp edge where in-place-mutation optimizations can
vanish on minor code changes.

The core thesis is that proof irrelevance (a correct proof's internals don't
matter, only its existence) combined with LLM-generated proofs represents "an
extremely capable form of proof automation." He declines to publish his code,
suggesting LLMs could likely do better than he did on a problem this
well-defined. An aside on verified assembly (AWS's LNSym for AArch64) notes
that equivalence proofs between optimized assembly and Lean are possible for
tiny functions but didn't scale in his experiments.
