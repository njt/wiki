# Sunflowers — Grading Difficulty by the Logic a Grid Forces

Sunflowers is a flower-planting logic puzzle whose site documents how it grades difficulty: rather than heuristics or vibes, every grid is solved end-to-end by the same engine that powers the in-game hints, always taking the simplest move like a careful human, and the tier is named for the hardest technique the grid *forces*.

---

The five techniques ladder up cleanly: single-clue deduction (Easy), clue subtraction (Medium), min/max bounds on overlapping clue regions (Hard), what-if contradiction chains (Extreme), and either-way colouring pivots beyond the chain cap (Insane). The what-if is capped at a few counting steps so every chain is "mentally trackable and no pencil marks are needed" — a design constraint, not an accident.

> "Every grid you generate is solved end-to-end by the same logic engine that powers the in-game hints... always taking the *simplest* move available, exactly like a careful human, and records which reasoning techniques the grid forced it to use."

This is the whole thesis. Difficulty isn't a property of the grid's density or size; it's the transcript of a canonical solver reading the grid. Because the solver is the hint engine, the difficulty label and the help a struggling player gets are guaranteed to agree — the game cannot lie about what it demands, because the same machinery proves it.

> "This is the one tier where keeping notes genuinely helps. Hints spell out both kinds: which square to test, and (for either-way) which squares both chains agree on."

Even the 0 tiles — the dull squares — are governed by `ZERO_CLUE_RULES` per tier, with the generator protecting, stripping, or injecting them. Nothing is left to chance; the difficulty contract is enforced on every axis.

**Key themes:** #pattern #concept #tool

## Take

This is a small masterpiece of verifiable design, and it deserves attention well beyond puzzle games. The move that elevates it: the difficulty tiers are *falsifiable contracts*. "A Medium grid must force at least this many two-clue subset moves" is a testable claim, and the sliders are the exposed test harness — each click tries up to 90 candidates, and if nothing qualifies, the closest match is served *and flagged as a compromise* rather than silently mislabelled. Compare that to the usual software practice of labelling things Hard because a number went up.

There is also a deep lesson in making the grader and the guide the same engine. Any system that claims to assess work should be the same system that helps you do the work — it's the trick that keeps assessment honest and makes the explanation free. Swap "grid" for "pull request" and you have an argument about review tooling: measure with the artifact you hand the human, not a parallel score nobody can interrogate.

The 0–100 score is the honest bit most ratings lack: non-overlapping bands, positioned by sub-metrics specific to the tier (solve width for Easy, forced-move counts for Medium/Hard, total chain *depth* rather than count for Extreme — since a 1-step what-if "plays much like an overlap move while a 4-step cascade is a different beast"). They even publish estimated-time weights in `EST_TIME` so drift from real solve times can be corrected in the open. Calibration as a public artifact.

## Related

This strengthens [[Solving Wordle with Information Theory]] in the same spirit: both treat puzzle difficulty as a measurable, information-theoretic or proof-theoretic quantity rather than a designer's hunch. It nuances [[The Awesome Power of Constraint Solvers]] — here a solver is not just a solution-finder but the instrument that *classifies* an instance by the reasoning it demands. And it echoes [[Proving It Works]]: the same machinery that grades the artifact verifies it, leaving no gap between claim and evidence.

---
*Sources: [[raw/sunflowersgame-com]], [[summary/sunflowersgame-com]]*
*Last updated: 2026-10-11*
