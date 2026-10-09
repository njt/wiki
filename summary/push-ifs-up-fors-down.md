---
url: https://debasishg.github.io/blog/push-ifs-up-fors-down/
title: "Push Ifs Up and Fors Down: The Idiom, Its Algebra, and Its Limits"
author: Debasish Ghosh
date_fetched: 2026-10-09
date_published: 2026-10-09
topics:
  - software-engineering-craft
  - databases-and-data
---

Ghosh takes the TigerBeetle "Tiger Style" heuristic — *"push ifs up and fors down"* — popularised in matklad's blog and gives it an algebraic reading. Pushing conditionals up means the caller owns branching: instead of `frobnicate(walrus: Option<Walrus>)` unpacking the option internally, the caller handles `None` and the callee takes a plain `Walrus`, so the type states the precondition. Pushing loops down means offering `frobnicate_batch(walruses)` so the hot loop is branch-free and vectorisable; the two moves compose via `filter_map`.

The essay then shows the same shape in two distant domains. In relational query optimisation, the vocabulary is inverted ("push a predicate down" means run it early), but the principle is identical: selections and projections execute before joins so expensive combining operators see the smallest inputs, and vectorised batch execution is literally `frobnicate_batch` at engine level — per-call overhead paid once per batch of ~1000 tuples instead of per tuple in Volcano-style row iteration.

The category-theory section is the heart: pushing an if up is restricting to a subobject — the callee's input becomes the subset `{a ∈ A | p a}` injected into `A` by a monomorphism. `Option<Walrus>` is the coproduct `1 + Walrus`, and a function branching on it is a function out of a coproduct which, by the universal property, is exactly a pair of functions; pushing the if up factors the pair apart. For filter/map, Ghosh proves the law `filter p . map f == map f . filter (p . f)` from the naturality of `catMaybes`, then notes the honest caveat: the rewrite only saves work when `p . f` reduces to a cheap predicate on the input — both sides are still O(n), you save calls to `f` on doomed elements.

The summary is the discipline the essay earns: every such rewrite is legal only under a constraint — loop-invariant conditions can leave a loop, per-element ones cannot; push-down of a selection below a join is valid only when the predicate references one side. "It's the algebra that tells you which rewrites are legal."
