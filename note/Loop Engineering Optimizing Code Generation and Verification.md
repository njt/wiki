# Loop Engineering: Optimizing Code Generation and Verification (Bradley Gibbs)

Bradley Gibbs offers the most disciplined treatment yet of the question every AI-coding workflow eventually hits: agents can generate code far faster than humans can verify it, so how do you design the process so human attention and judgment go only where mechanical evidence runs out? His answer is a three-loop architecture — an outer loop of human-approved conformance contracts and final merge decisions, an independent evaluator in the middle, and an engineered inner build loop whose errors are prevented, observed and corrected — plus a learning loop that treats every check as a measurable mechanism.

---

## Key quotes

> "The builder's completion message does not determine qualification, and missing evidence cannot count as a pass."

The load-bearing sentence. Self-reported completion is the default failure mode of agent harnesses; Gibbs makes independence of the evaluator a structural property, not a prompt. Compare [[Poor Man's Loop Engineering]], which reaches the same place with two ingredients.

> "A check that passes the intended implementation but also passes a defective one establishes little."

Check quality, not check quantity. The storage-read example drives it home: a candidate that reports the error but overwrites the data satisfies half the obligation — the checks must exercise both halves, and be run against a deliberately defective implementation to prove they can fail.

> "Treat 57.9% as evidence that mishandling available information is common and worth examining, not as a rate we can apply to our own loop."

The *Failure as a Process* finding (most decisive agent errors are epistemic, not competence) is quoted and then immediately dismantled as a transferable statistic: different definitions, one decisive error per run versus several failure modes, no accounting for recoverability. This is rare epistemic hygiene — most secondary citations in this genre launder exactly this kind of number.

> "With perfect terminal evaluation, early checks add no correctness guarantee to qualified code."

The thought experiment that disciplines the whole inner-loop argument: under a perfect evaluator, inner-loop engineering buys time, tokens and rework reduction — not correctness. False greens only enter the picture when the perfect-evaluation assumption is relaxed. Most harness engineering writing conflates these two motivations; Gibbs keeps them separate.

> "A shorter review is useful when it preserves or improves the quality of the decision."

Against raw speed metrics: a faster run that hands reviewers a less intelligible candidate has *moved* the burden, not reduced it. The evidence package itself must be measured.

## Key themes

#concept #pattern #tool

- **Conformance contract as the boundary object.** Humans and AI cooperatively translate intent into obligations (do, preserve, never permit) before the run; the contract is fixed during a bounded run — the agent may suggest changes but never silently alter an obligation or check.
- **Environmental errors vs agent failure modes.** Missing prerequisites get prevention (stage design, handoff discipline); agent mistakes despite adequate information get observation, targeted checks, bounded repair. The unfaithful-handoff walkthrough (TD-08, the overwritten store) shows an obligation lost in an intermediate artifact, not by the source contract.
- **Checks and corrections as rates.** Detection rate, correction rate, cost per invocation, consequence of escape — no universal threshold; mechanisms earn or lose their place empirically, and must be requalified when the model or harness changes ("the model eating the harness" gets a concrete test).
- **Diagnostic resolution.** Terminal verdicts are black boxes; a pass hides whether the run was clean or error-riddled-with-recovery, and the wasted work difference is exactly what inner-loop engineering targets.

## Opinionated take

This is the anti-hype pole of the loop-engineering literature, and it is better for it. Where most harness writing is a confident taxonomy plus one triumphant metric, Gibbs builds the taxonomy and then spends pages explaining why his own evidence can't yet support the claims the genre wants to make — no measured savings reported, walkthroughs labelled illustrative, arithmetic on check overhead (20 seconds × 100 invocations = 33 minutes before any repair) that punctures the "early detection always wins" intuition. The cost is readability: the hedging is relentless and the piece runs long. But the structure survives the honesty — the independent evaluator, the fixed contract, the obligation-indexed review packet, and the disabled-condition experiment for deciding whether the model has "eaten" a harness mechanism are all genuinely reusable designs. The weak point is that the method is expensive by construction: this is a platform team's workflow, and nothing here tells a solo developer what the minimum viable version is. Its strongest claim — that deterministic verification should *direct* human attention rather than replace it — is the right frame for the whole review-bottleneck debate.

## Relations

- **[[Poor Man's Loop Engineering]]** — the same thesis (test your own work, independent review) at minimum scale; Gibbs' piece strengthens it by supplying the full formal architecture the poor man's version approximates, and nuances it by showing how much machinery lives between "two cross-model reviewers" and an auditable pipeline.
- **[[Own the Outer Loop]]** — Gibbs explicitly builds on Osmani's human-at-the-boundaries framing (the article cites "Own the Outer Loop" for cooperative contract-writing); this source turns that essay's accountability rhetoric into concrete mechanism: obligations, evaluator independence, review packets.
- **[[The End of Code Review]]** — Monperrus argues mandatory human review is indefensible; Gibbs complicates that by relocating rather than removing human judgment — the reviewer only sees claims mechanical checks couldn't settle, which is a third way between rubber-stamping and full autonomy.
- **[[Harness Engineering in Practice]]** — shares the harness-as-optimization-target idea (Gibbs cites Weng's survey in the same spirit); Gibbs adds the learning loop's empirical discipline: keep, narrow, retire, requalify as the model changes.

---
*Sources: [[raw/loop-optimization]], [[summary/loop-optimization]]*
*Last updated: 2026-10-12*
