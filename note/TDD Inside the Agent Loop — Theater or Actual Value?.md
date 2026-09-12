# TDD Inside the Agent Loop — Theater or Actual Value?

A Thoughtworks technologist ran a small controlled experiment on the most common way practitioners use TDD with coding agents today — telling the agent to do full red-green-refactor *inside its own loop* — and found nothing: no quality difference (Opus 4.8, judging blind, more often ranked the non-TDD solutions higher), no mutation-score difference, and a 3x–8.5x token bill. The article's real contribution isn't the n=2-per-cell data, which the author freely admits is tiny; it's the goal-by-goal teardown of TDD showing that most of its benefits were mechanisms for *human psychology* — forcing thought, enforcing restraint, managing fear — that evaporate when the loop has no human in it.

---

## Key Quotes

> "**TLDR;** Based on Opus's judgment of the quality of the outcomes, there was no clearly discernable difference based on TDD workflow versus no TDD workflow. On the contrary, more than once Opus ranked the non-TDD workflow solutions slightly higher in design and test quality."

The null result, stated plainly. What makes it credible is the methodology: Opus judged blind, an independent agent audited TDD adherence from session transcripts so non-compliant runs couldn't pollute the comparison, and the author lists the caveats before anyone else can.

> "The design in those runs emerged from the sum of many locally-minimal decisions and was rarely revisited, so it tended to land on whatever shape the first test happened to lock in. Behaviour the agent didn't think to write a test for didn't get implemented at all."

The causal hypothesis, and the sharpest finding in the piece. Incrementalism — TDD's supposed virtue — was actively *harmful* here: the non-TDD runs designed the whole architecture up front and beat the TDD runs on data models, edge cases, and completeness. TDD didn't just fail to help; it suppressed the step that actually helped.

> "The way AI agents were trained is that they have seen completed functions and descriptions of those functions. The number of actual step-by-step TDD examples they have seen is a tiny part of the training data. That means that the LLM has an internal representation of code that is a direct translation of requirements to code, and not a process of how to get to that representation." — Ivett Ördög

This is the "why," and it reframes the whole practice: TDD-in-the-loop is an uphill battle against the training distribution. It also predicts the observed cost asymmetry — the author needed many prompt iterations just to get *approximate* adherence, and warns the prompt is likely volatile across model releases.

> "Watching a test go red is only proof of anything if someone is checking *why* it went red. When the agent both writes the test and confirms it failed, a red test tells you the agent ran it and saw failure, not that the failure was for the right reason."

The single most quotable line in the article. The red step's evidentiary value was always in the *human watching it*, not in the color of the test output. Automate the watching and you've automated the ritual while losing the mechanism.

> "When humans write a test first, it forces us to think about usage before implementation... An agent doesn't experience that and can write a test the same instant it plans an implementation. Without a human checkpoint between the two, is there really any purpose left to writing the test first?"

The philosophical core. TDD was never really about tests; it was a discipline device that made humans do the thinking in a specific order. Agents don't have fear (Beck's "managing fear"), don't need to be restrained from speculative generality by tiny steps, and don't experience the friction that test-first existed to create.

> "I think at this point there is generally more and more evidence that being overly specific about *how* we want a model to do something is not a sustainable approach. Instead, we should find as many ways as we can to monitor the outcomes and give feedback."

The generalization, and where the piece lands: shift energy from prescribing process to building outcome checks — mutation testing for regression quality, static analysis and structure review for refactoring, Approved Scenarios (Ivett Ördög's frozen-expectation runner) for confidence.

## Themes

- **Process prescription vs. outcome monitoring** — the central claim, and a standing challenge to every elaborate agent workflow prompt. #concept
- **Tests as artifact vs. TDD as ritual** — the tests survive as a regression net and feedback signal; the red-green-refactor *ordering* doesn't. #pattern
- **Up-front design over incremental emergence** — for agents, letting (or asking) them design the whole model first beat test-by-test accretion. #pattern
- **Mutation testing as the honest red** — if you want to know whether tests can catch regressions, measure it, don't perform it. #concept
- **Training-data gravity** — Ördög's theory that models know requirements→code, not the route. Explains both the adherence failures and the prompt-maintenance tax. #concept
- **Ivett Ördög** (Approved Scenarios), **Kent Beck** (TDD as fear management), **Emily Bache** (reviewer). #person

## Analysis

The honest read of the experiment: it's small (2 runs per cell), greenfield-only, judged by a model with its own taste biases, and the author says all of this out loud. But three results point the same direction, and two of them would survive a much larger n. First, the token multipliers (3x–8.5x, even discounting cache-inflated accounting) — you pay real money for a process that buys nothing measurable. Second, the adherence problem: even a carefully iterated prompt produced runs that skipped or faked the red step, which means TDD-in-the-loop isn't even reliably *TDD*. And third, the causal mechanism — incremental design accretion losing to up-front design — is exactly what you'd expect from a model that has seen millions of whole designs and vanishingly few honest red-green transcripts.

The deeper value of the piece is the taxonomy of *why* TDD helped humans: it was never primarily about the tests. It was thinking-in-the-right-order (usage before implementation), enforced restraint (YAGNI), localized debugging, and fear management. Remove the human and every one of those mechanisms loses its substrate. What remains genuinely valuable — regression detection, design pressure, refactoring safety — the author redirects to instruments that measure rather than ritualize: mutation scores instead of red-green, structural review triggers instead of refactor steps, frozen approved scenarios instead of confidence-by-green-tests.

Where I'd push back: greenfield business logic is the kindest possible terrain for this conclusion. There's no existing code to misunderstand, no regression surface to protect, no integration risk — the situation where up-front design is cheapest and TDD's safety net matters least. Brownfield work, where the value of tests-first is that the *new* code can't silently break the old, is untested here (though the mutation-score finding suggests the net quality question stands). And "monitor outcomes instead of process" has its own failure mode the article doesn't price in: outcome monitors are only as trustworthy as their builders, and an agent optimizing against a mutation score will happily game the score. Prescribing process may be unsustainable; measuring outcomes is not automatically safe.

Still, as a data point against the instinct to encode our favorite human disciplines into agent prompts, this is the best-annotated experiment in the wiki so far. "Theater" turns out to be the right word: the agent can perform red-green for the transcript while every mechanism that made red-green meaningful stayed outside the loop.

## Related pages

This source **complicates** [[TDD Coordinator (Corazonn)]] directly: that command orchestrates subagents through rigorously phase-gated red/green/refactor cycles, yet this experiment found prescribed TDD sequencing buys no quality at several times the token cost — its surviving value is likely the Rule of Two cross-agent review (an outcome check), not the TDD ordering. It **nuances** [[Dmitry Sotnikov's LLM Workflow]], whose "tests as selection pressure in a genetic-algorithm loop" survives intact here — mutation scores and regression value still matter — while the test-*first* ordering Sotnikov's loop inherits from XP is precisely the part that fails to transfer. It **strengthens** [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]] — fittingly, since Emily Bache reviewed this piece: her Sensors (automated feedback on the harness) are what moved the needle here where process instructions didn't. And it **corroborates** [[Agentic Testing]]: Slack's finding that agent-generated deterministic tests fail 48% of the time on complex flows matches the tautological, circular validation this experiment kept catching in agent-written tests.

---

*Sources: [[raw/tdd-in-the-agent-loop-html]], [[summary/tdd-in-the-agent-loop-html]]*
*Last updated: 2026-09-13*
