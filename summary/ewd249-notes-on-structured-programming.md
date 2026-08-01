---
url: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD249.PDF
title: "Notes on Structured Programming"
author: Prof.dr. Edsger W. Dijkstra
date_fetched: 2026-07-18
date_published: 1970-04
---

Dijkstra's foundational monograph on structured programming, originally written in 1969 and published as a T.H. Eindhoven technical report. It is the document that introduced the term "structured programming" and laid out its intellectual foundations.

The core argument: humans cannot hold the full state of a large program in their heads. Our only hope is to construct programs in a hierarchical, provably correct way — building each level out of components already known to be correct — rather than writing a monolithic program and attempting to debug it into working order.

Dijkstra attacks the prevailing "test and patch" approach. Testing can show the presence of bugs, never their absence. The only reliable approach is to prove programs correct as they are constructed. He introduces the control structures that enable this — concatenation, selection, and repetition — and shows how their properties support inductive correctness proofs.

The monograph introduces three enduring ideas. **Step-wise refinement**: start with an abstract program and decompose it into more concrete steps, proving each decomposition correct as you go. **Layered virtual machines**: build software as a stack of abstract machines, each providing a cleaner conceptual interface to the one above it. **Program families**: design programs so that different versions can be derived from a common, well-understood ancestor rather than built from scratch.

Dijkstra illustrates his method with two extended examples: generating a table of the first thousand primes, and a line-printer plotting program. Throughout, he emphasises that intellectual manageability — not raw performance — is the true constraint on what we can build.

The "pearls and necklace" metaphor closes the work: a well-structured program is a string of independent, correct sub-programs (pearls) connected by simple control flow (the string). Understanding becomes linear: read each pearl once, in order, and you understand the whole.

---
*Sources: [[raw/ewd249-notes-on-structured-programming]]*
*Last updated: 2026-08-01*
