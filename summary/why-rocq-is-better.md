---
url: https://joomy.korkutblech.com/posts/2026-07-28-why-rocq-is-better.html
title: "Why Rocq is better than Lean for program verification"
author: Joomy Korkut
date_fetched: 2026-08-01
date_published: 2026-07-28
topics:
  - software-engineering-craft
---

Joomy Korkut explains why he continues to use Rocq rather than switching to Lean for formal program verification, despite Lean's momentum in mathematics and AI-assisted proof. The "better" in the title is deliberately provocative — he means "a better fit for my work today."

The core technical argument rests on **coinductive types and cofixpoints**. Rocq supports `CoInductive` and `CoFixpoint` natively, letting him declare codata, write guarded recursive producers, reason about them by observation, and extract them to direct lazy OCaml code. Lean has no equivalent kernel declarations. Its alternatives — `Stream'` (just `Nat → α`), `Iter`, `Thunk`, `partial def` — each trade off some combination of expressiveness, proof transparency, or extractability. Alex Keizer's QPFTypes proof-of-concept comes closest to Rocq-style codata in Lean but fails on parameterless, mutual, and indexed coinductive declarations that are routine in Rocq. This matters for interaction trees, which represent effectful, nonterminating programs as coinductive trees and underpin much of Rocq's verification ecosystem.

A second technical gap is **nested inductive types**: Lean's kernel rejects some definitions that Rocq accepts, such as a validation relation combining `Forall₂` and `And`. Lean users can split the relation or introduce mutual definitions, but this adds proof machinery.

On **program extraction**, Rocq offers multiple backends — OCaml, Haskell, Scheme, Rust, Elm, C++, Clight/WebAssembly via a verified compiler — while Lean compiles through a single opinionated runtime pipeline with no end-to-end correctness proof.

The **ecosystem** section catalogs Rocq's verification infrastructure: Iris (concurrent separation logic), VST (verified C), CompCert (verified C compiler), interaction trees and choice trees, Fiat Crypto, and lighter-weight translation tools for Haskell, OCaml, Python, and Rust. Many of these are battle-tested and hard to replicate.

Korkut also notes that Rocq has **regulatory acceptance**: ANSSI has criteria for using it in Common Criteria evaluations, and CompCert was qualified for avionics in 2026.

On **AI agents**: he dismisses the concern that AI only writes Lean, noting that current models handle Rocq well given its large corpus, and language popularity is a short-lived argument.

The post concludes that Lean is doing serious program-verification work too, but porting his work would mean reshaping definitions and replacing pipelines, libraries, and institutional history he already relies on. For his actual work, Rocq remains the better fit.
