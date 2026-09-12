# Kale — Transformation-Safe Spreadsheets

Kale is a research spreadsheet system that eliminates a whole class of bugs — the ones introduced when a user restructures a table and formulas silently change what they point at — by making the dangerous references unexpressible. Instead of detecting errors after the fact, Kale restricts formulas to single cells, whole columns, and whole rows, then redefines "absolute" and "relative" so that references survive insertion, deletion, and sorting. It is the spreadsheet analogue of the "make illegal states unrepresentable" idea: you can't write the buggy reference, so you can't ship it.

---

## The Core Diagnosis

> "Do spreadsheet references actually refer to the *data* in a cell, or do they refer to the *geometry* of the cell?"

This is the question the whole paper turns on, and it's sharper than it looks. Traditional spreadsheets answer inconsistently: insert a row above `B2` and the reference follows the *data* down to `B3`; drag cell `C3` away from `SUM(B2:C3)` and the reference stays glued to *geometry* while its value silently changes. Kale's answer is that references should always track the data, and the way to guarantee that is to forbid the reference forms — arbitrary rectangular ranges — where the two interpretations can diverge.

> "Confusingly, although traditional spreadsheets distinguish between *relative* and *absolute* references, this distinction does not address risks R1-R4."

The authors' most damning empirical point. Every spreadsheet user has learned the `$A$1` convention, but that distinction only governs what happens when you *copy* a formula — it does nothing when you *restructure the referenced table*. So users carry a mental model of "absolute = safe" that is simply wrong for the operations that actually corrupt data. Kale redefines the terms to mean what users already believe they mean: absolute = follows the data, relative = stays at the same offset.

## The Five Risks

The paper catalogs the failure modes as R1–R5, each a specific way a structural edit breaks a formula:

- **R1** — inserting a row/column adjacent to a range: should the range grow? Spreadsheets say no (with Excel emitting a warning nobody sees).
- **R2** — sorting permutes rows without updating references.
- **R3** — moving cells makes ranges silently expand or contract.
- **R4** — named cells don't follow their data through a sort.
- **R5** — relative references are the copy/paste default, so drag-fill can grab the wrong cell.

> "The rates at which participants using Sheets inserted errors due to these risks (50-83% for R1, 56% for R2, 30% for R3, and 78% for R4) were orders of magnitude larger than Panko's estimated global cell error rate."

This is the finding that makes the paper matter beyond its own prototype. The error rates Kale targets aren't a rounding error on the usual 1–5% cell-error estimates — they're catastrophic, and they're invisible: a user inserts a row, the formula keeps evaluating, and nothing turns red. The 47% success rate on the preliminary survey shows users don't even have a correct mental model to fall back on. The bugs are silent, structural, and nearly universal.

## Key Themes

- #concept **Reference instability** — references are unstable under structural changes; data-vs-geometry is the ambiguity at the root
- #tool **Kale** — prototype spreadsheet (TypeScript/React + AG Grid) that guarantees preservation of referenced data through transformations
- #pattern **Safety by construction** — make the dangerous expression unrepresentable rather than detecting it; the spreadsheet form of [[Parse Don't Validate]]
- #concept **Bounded cognition in end-user programming** — users can't track latent reference relationships through structural edits, so the system must remove the burden

## Critical Analysis

**The one-idea paper done right.** Kale is a thin prototype, but the paper's contribution isn't the system — it's naming and measuring a bug class that the 95%-of-spreadsheets-are-wrong literature had never isolated. Prior work focused on *entering* wrong formulas; the authors show a large share of errors come from formulas that were *correct* until a later edit broke them. That reframe is the durable claim.

**The tension the authors don't fully resolve is with expressiveness.** The corpus study — all 60 sampled EUSES spreadsheets ported successfully — is the paper's weakest link. Converting a spreadsheet by hand in 18 minutes is not the same as a user *authoring* it under Kale's constraints; the authors admit the sampled set excluded cross-table references (only 3% of the corpus, but likely concentrated in the most sophisticated workbooks). The real question isn't "can an expert port it" but "can a naive user express what they mean," and that's left for future work.

**The sharpest contrast is with [[Grist]].** Both systems break from the traditional grid, but in opposite directions: Grist maximizes formula expressiveness (full Python) and accepts the resulting complexity; Kale minimizes it and accepts the loss of rectangular ranges. That's the design fork the paper opens — is the right response to spreadsheet error *more* power or *less*? Kale's evidence — 50–83% error rates on tasks naive users actually perform — is a serious argument for less.

**The connection to [[Engineering for Bounded Cognition]] is the deepest.** Kale is, at bottom, a prosthetic-cognition argument: users cannot hold the latent graph of "which formulas point at which rows" in working memory while they insert a row, so the system must make it impossible for that gap to produce wrong answers. The authors even show the warning-based alternative fails — Excel emits a warning on R1-style insertions, and users' error rates show it may as well be a decoration.

**The honest weakness is R3.** In two Kale cases, the automatic "follow the data" behavior *introduced* a new error: participants swapping values by moving rows found Kale silently preserved the references they expected to swap. This is the mirror-image failure — safety-by-construction makes the system's implicit choice the author's intent, and when that intent is wrong, there's no error to notice. It's the same tradeoff [[Parse Don't Validate]] warns about: encoding an invariant in the structure works only when the invariant is the one you wanted.

---

*Sources: [[raw/2608-26345v1]], [[summary/2608-26345v1]]*
*Last updated: 2026-09-08*
