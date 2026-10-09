# Push Ifs Up and Fors Down — The Algebra of Code Motion

Debasish Ghosh takes the TigerBeetle/matklad idiom "push ifs up and fors down" — centralise branching in the caller, keep loops branch-free in the callee — and grounds it in database query optimisation and category theory, arriving at the claim that *the algebra tells you which rewrites are legal*.

---

## What it argues

Tiger Style's rule is usually read as taste: keep control flow in one function, move branch-free fragments to helpers. Ghosh upgrades it to a structural principle with a validity condition.

**Push ifs up** is about where the decision lives, not how much data flows downstream. `frobnicate(walrus: Option<Walrus>)` branching internally becomes a caller-side check plus `frobnicate(walrus: Walrus)` — the type now states the precondition.

**Push fors down** is about cost shape: `frobnicate_batch(&walruses)` pays setup once per batch and runs a tight, vectorisable inner loop. The two compose: `filter_map` the `None`s at the boundary, hand the plain vector down.

The two extended analogues:

- **Query optimisers** say the same thing upside-down. Because data flows up a plan tree from leaf scans to root, "push selection down" means run it *early* — the same direction Ghosh calls "up". Selections and projections below joins, plus vectorised batch execution over Volcano-style row-at-a-time `next()`, are `frobnicate` versus `frobnicate_batch` at engine scale.
- **Category theory** makes the if-up move exact. The subset `{a ∈ A | p a}` with its inclusion is a *subobject*; a function that takes `Option<Walrus>` (the coproduct `1 + Walrus`) and branches is a function out of a coproduct — which by the universal property *is* a pair of functions. Pushing the if up factors the pair apart: caller owns the `1` summand, callee is the `Walrus` component.

The filter/map section is the most careful. The naive advice "filter before you map" is not an equivalence in general — `filter p . map f` tests outputs, `map f . filter p` tests inputs. The real law is:

```
filter p . map f  ==  map f . filter (p . f)
```

derived from the naturality of `catMaybes` (filter itself is not a natural transformation; catMaybes is). And the punchline is honest: the rewrite is not automatically cheaper — `filter (p . f)` still computes `f` for every element. It pays off only when `p . f` simplifies to a cheap predicate on the input.

## Key quotes

> "All control flow should be handled by one function, the rest shouldn't care about control flow at all. In other words, 'push ifs up and fors down'."

Tiger Style's formulation — the idiom's origin, read as divide-and-conquer over responsibility rather than micro-optimisation.

> "In Set, a monomorphism is an injective function: the inclusion sends each element to itself... The callee no longer needs the `if`, because every input it can receive has already passed."

The strongest move in the piece: the type-refinement reading of if-up is not a metaphor, it is a subobject, and the code-level witness is `Walrus` instead of `Option<Walrus>`.

> "It's the algebra that tells you which rewrites are legal."

The summary's thesis, and the reason the essay is more than a style tip roundup: each rewrite carries a validity condition (loop-invariant condition, one-sided join predicate, simplifiable composed predicate), and the algebra is what separates legal code motion from silent semantic change.

## Themes

#concept #pattern #software-craft #algebra

## Analysis

What's genuinely good: Ghosh resists the temptation to inflate the idiom into a universal law. He states the constraints that bound it — a per-element condition *cannot* leave the loop, it can only move to the boundary and be recorded in a type; a pushed-down selection is legal only when the predicate references one side of the join; filter-before-map saves work only under a decomposability assumption. That restraint is rare in idiom essays and is exactly what makes the category-theory section load-bearing rather than decorative.

What's thin: the "fors down" half gets algebraic short shrift — Ghosh concedes it is "more about the cost than the equivalence," a change of arrow shape from `A -> B` to `[A] -> [B]`. That is a real asymmetry and the essay flags it rather than resolving it; a theory of when batch-ifying is worth it (cache behaviour, vectorisation, per-call overhead) is gestured at, not developed.

The database analogy is the most accessible of the three but also the most inexact — Ghosh works hard to reconcile the inverted vocabulary ("push down" in a plan tree = "up" in call order), which is honest but a bit of a strained seam.

The unstated relevance to the present wiki's main beat: this is exactly the kind of structural knowledge that separates code an agent *writes* from code an agent can *reason about and rewrite*. Rewrites whose legality is algebraic (not vibes) are precisely the transformations a model can apply safely when the types state preconditions. Type-state-refinement is an agent-legibility device.

## Related pages

This strengthens [[The Economic Benefit of Refactoring]]: Fowler's measured 83% token reduction from refactoring agent-generated code is downstream of exactly the kind of semantic decomposition Ghosh formalises — narrow the type, shrink what the next consumer must read.

This nuances [[Apache DataFusion]]: Ghosh's plan-tree discussion (selections/projections below joins, vectorised vs Volcano execution) is the theory-of-optimisation backdrop for DataFusion's actual rule-based optimiser; the inverted push-down vocabulary discussed here is that system's native idiom.

This complicates [[Simplicity in the Age of AI-Assisted]]: simplicity here is not minimal surface area but precisely stated preconditions — the "simple" function is the one whose type already contains the filter, which is a stricter demand than mere brevity.

This deepens [[Push Ifs Up And Fors Down]]: where that note records the idiom as stated (matklad's two heuristics, TigerBeetle as macro-scale example), Ghosh supplies the algebra beneath it — subobjects and coproducts for if-up, arrow-shape and naturality for the rest — turning a style rule into a set of provably legal rewrites with explicit validity conditions.

---
*Sources: [[raw/push-ifs-up-fors-down]], [[summary/push-ifs-up-fors-down]]*
*Last updated: 2026-10-09*
