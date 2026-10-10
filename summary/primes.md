---
url: https://blueberrywren.dev/blog/primes/
title: "In Which I Formalize the Infinitude of Primes in Four Theorem Provers"
author: blueberrywren
date_fetched: 2026-10-10
date_published: 2026-10-10
topics:
  - developer-tools
  - software-engineering-craft
---

blueberrywren formalizes Euclid's proof of the infinitude of primes in Lean, Isabelle/HOL, HOL4, and Agda, and compares the four theorem provers on user experience rather than expressiveness: automation quality, proof transparency, theorem search, interactivity, and the classical-vs-constructive divide.

The two foundational camps frame everything. Lean and Agda are dependent-type systems built on the Curry-Howard correspondence, where proofs are terms you carry around. Isabelle/HOL and HOL4 are LCF-style systems with a small trusted proof kernel — all proofs must be constructed from base rules, and classical logic (excluded middle, Hilbert's epsilon, the Axiom of Choice) comes for free. The author comes down firmly on the LCF side: proof terms feel like dead weight and classical logic is too useful to give up.

Automation is the biggest practical differentiator. Isabelle/HOL's `sledgehammer` calls external SAT/SMT solvers and can often close "doable-but-annoying" goals outright (at the cost of indecipherable proofs); HOL4 has `metis_tac` and rich simplification. Lean's `grind` is unpredictable — "sometimes smarter than `sledgehammer` and sometimes stupider than `simp`" — and both `sledgehammer` and `grind` are all-or-nothing, whereas partial automation like `simp` or `auto` makes progress and leaves the context better. Agda has essentially no automation, which "sort of sucks" — but constructivity pays a dividend: the prime proof is executable, so it can actually *generate* a prime larger than any given n (horribly slowly, checking primality near 360,000 inside `(n+1)!+1`).

Tooling around the proofs matters as much as the proofs. Isabelle/HOL and HOL4 have excellent pattern-based theorem search (`find_theorems`, `DB.match`) — "you know the *shape* of something you want" — while Lean's leansearch/loogle are weaker and Agda offers nothing but library paging. Interaction models diverge too: Lean wants VSCode, Isabelle ships jEdit, HOL4 works through an Emacs/Vim bridge to a REPL, and Agda lives entirely in Emacs with case-split keybinds. Notably, only Lean and Isabelle/HOL show proofs in progress; HOL4 and Agda proofs are completed terms you must disassemble.

The verdict, on the author's own five arbitrary criteria, crowns HOL4 ("I had a great time learning it, it's a seriously interesting system"), with Lean close but "slightly wary"-inducing, Isabelle/HOL familiar and fine, and Agda last for its manual drudgery. The closing advice: try the camp you haven't used.
