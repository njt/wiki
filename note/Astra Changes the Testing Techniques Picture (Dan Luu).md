# Astra Changes the Testing Techniques Picture (Dan Luu)

Dan Luu re-runs his test/verification-technique evals with GPT-6 Astra and finds the prior picture partially inverted: Astra saturates the old Zstd benchmark, and on a much harder Pandoc eval ($90k of tokens) many prescribed techniques actually beat Default prompting — the opposite of 5.6 Sol, where technique prescriptions were mostly a caricature of the real thing. The post is as much about epistemic limits as results: Luu is scrupulous about what $90k can and cannot tell you.

---

## What the eval found

- **Zstd is saturated.** 5.6 Sol spent $3.50–$5.50 to hit 63–77% holdout correctness; Astra spends ~$1 to hit 99%+ at medium effort. A benchmark agents ace on every condition stops being a benchmark.
- **Pandoc is the new frontier.** Even at ~14 turns average, no condition reached 100% correctness, so Luu measures fraction of tests passed across 25 runs per condition.
- **Many conditions now beat Default.** In 5.6 Sol, no technique clearly outperformed doing nothing special; with Astra, enough do that "one or more conditions truly outperforms Default" is plausible — even though we can't name which.
- **Specific interesting signals:** differential testing got agents to genuinely examine boundary conditions; TDD underperformed again (as Luu predicted); ACL2, the underrated theorem prover, performed well again; property-based testing libraries finally produced tests that behave like real PBT.

## Key quotes

> "not specifying any particular test method (which we'll call Default) resulted in many worthless unit tests, which was one of the best performing conditions because asking agents to test in specific ways mostly made them worse."

The baseline against which the new results must be read. Technique prescriptions used to be a tax.

> "nice-sounding explanations for why something ought to be better that sound appealing in a very human way but appear to be unrelated to reality when we look at what agents actually did"

Luu's recurring thesis: post-hoc rationalizations for tool choices (including for humans — he notes the static-vs-dynamic-types literature is equally threadbare) don't survive contact with measured agent behavior.

> "if you keep in mind that we don't know how well any of these results generalize, I'll note a few things that I thought were interesting."

The whole post in one sentence. He wants 100x the runs to answer his own questions about path dependence, prompt sensitivity, and whether one Pandoc-sized problem represents anything.

> "AI-written unit tests and integration tests are empirically proven to be unhelpful"

The viral DeepSWE claim he dismantles: a 9% cost saving on easy problems, run on a grader that literally penalizes adding tests (an agent can collide with a secret test name). "Making overstated claims that something is empirically proven is a good way to go viral... but it's not a good way to figure out how to use coding agents effectively."

## Key themes

#concept #eval #pattern #person

- **Capability changes the prescription.** Techniques that fail with weak models may work with strong ones (and vice versa) — advice doesn't generalize across model generations.
- **Caricature → competence.** Weak models perform a hollow imitation of TDD/PBT/fuzzing; Astra's PBT and fuzzing actually exercise interesting code paths. Luu guesses this emerges from general capability, not targeted improvement — which would mean testing keeps getting better for free.
- **Epistemic humility as method.** Bootstrap intervals understate conceptual error bars; the honest claim is directional, not categorical.
- **Skills don't port.** MacIver's Hegel skill improved for one model/effort and got worse for another; test-technique skills are hard to write across models.

## Opinionated take

This is Dan Luu at his best: someone who actually spends $90k of his own tokens on a question everyone else answers by vibes, then tells you precisely how little the answer means. The meta-lesson matters more than any single result — the field's loudest empirical claims (the DeepSWE tweet) are built on broken graders and easy benchmarks, while the careful work explicitly refuses to overclaim. The ACL2 result is quietly significant: while the industry pours effort into Lean, a decades-old theorem prover keeps winning evals it wasn't tuned for, which is exactly the kind of signal only unbiased measurement can surface. The Leventov appendix (300k lines of generated test harness yielding 10% of found bugs) is a valuable counterweight against reading the main results as "generated tests are great" — cost-effectiveness of the testing apparatus itself is a separate axis from correctness gains. The gap Luu names — labs could answer these questions and don't — remains the scandal of the field.

## Related pages

This strengthens [[Agentic Verification]]'s claim that verification loops are where the leverage is — and adds a crucial nuance: which verification technique wins depends on model generation and problem difficulty, so the loop must be re-tuned as models improve. It independently corroborates [[TDD Inside the Agent Loop — Theater or Actual Value?]], where prescribed TDD again showed no quality benefit — though Luu complicates that page's story, since other small-test-heavy conditions did *not* underperform this time. It extends [[Dan Luu on AI Coding]]'s running argument that plausible-sounding rationales for tool choices don't survive measurement, now with the strongest evidence yet. And it connects to [[GPT-6 Astra Ascends NetHack]] as another data point on Astra's capability jump — there in game-playing, here in the economics of verification.

---
*Sources: [[raw/astra-testing]], [[summary/astra-testing]]*
*Last updated: 2026-11-20*
