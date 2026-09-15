# What Matters After the Software Factory Works

A six-month field report from someone who built an autonomous software factory in February, let it merge around 1,700 pull requests by June, and then wrote the memoir of what actually mattered — which turns out to be everything the factory's architecture diagram leaves out. The piece ends by naming a governance model, PAAA (Purpose, Articles, Actors, Artifacts), that the author claims explains both where the factory succeeded and where it wobbled.

---

## The argument in one paragraph

The claim: once a software factory's loop works, its success is determined not by the machinery — which is converging across the industry and becoming a configurable product — but by four governance properties that no platform ships: enforced rules (Articles), calibrated named actors (Actors), preserved decisions and evidence (Artifacts), and a human-held monopoly on direction (Purpose). This is falsifiable: if the author is wrong, then factories with converged architectures but no deliberate governance should perform comparably over six months, prompt-engineering alone should have produced trustworthy output, and self-generated agent work should remain valuable when human intent goes quiet. The author's own two-week silence experiment, in which the factory "hunted" and optimized noise, is the strongest evidence offered that the last of these is true.

## Key quotes

> **A plausible change nobody has verified is not velocity. It is inventory.**

The sharpest sentence in the piece, and a direct jab at the genre it belongs to — including the author's own earlier posts. It reframes the headline metric of every factory write-up as a liability accumulated before measurement exists.

> A rule without an enforcing check is an opinion, and a factory will politely route around opinions.

This is the conformance thesis in miniature: instructions to agents are suggestions, and only mechanically checked rules bind. It aligns with the fail-closed gate philosophy elsewhere in the wiki, but adds the failure mode — polite routing around — which is more insidious than open defiance.

> Named seats you can calibrate beat anonymous runs you can only rerun.

The least expected lesson. The author's discovery that reviewer seats themselves need performance ledgers — that "some reviewers agreed with everything, which is a defect, not a virtue" — extends calibration from outputs to the judges of outputs, a second-order move most harness writing has not made.

> When every run has to re-learn the codebase's quirks, you do not have a factory. You have an expensive amnesiac.

The memory lesson stated as an economic test: month six should be cheaper than month one, and the difference is memory. This gives institutional memory a measurable definition rather than a vibes-based one.

> What it cannot reliably do is decide *what is worth wanting.*

The conclusion of the quiet-fortnight story, and the piece's answer to where the human sits in an autonomous pipeline: not reviewing code, not even reviewing evidence, but upstream of everything as the sole source of direction.

## Critical analysis

What is genuinely non-obvious here is the calibration-of-judges move. The harness canon has converged on adversarial review and fail-closed gates; this piece goes one layer up and asks whether the adversarial reviewers themselves are any good, answering with ledgers of whether each seat's findings survived scrutiny and retrospective re-checking of old verdicts. That is a practice, not a slogan, and it is rare in the literature. The quiet-fortnight anecdote is also valuable precisely because nothing broke: the system stayed healthy while drifting purposeless, which is a failure mode invisible to every observability dashboard aimed at "is it running" rather than "is it running toward anything."

The weaknesses are the weaknesses of the memoir genre. This is n=1, on an unnamed codebase, with no numbers beyond the PR count — no defect escape rate, no cost curve, no measure of how much human time the seats consumed. The claim that "the machinery is becoming a product" and that platforms will absorb the loop is asserted, not argued, and it conveniently positions the author's remaining contribution (governance) as the durable part. PAAA itself risks being a retrofitted acronym: the Tuesday-example is well chosen, but "closed under its own governance" is stated in one sentence and never tested — the hard question of whether agents should be able to ratify changes to the Articles that constrain them is raised and immediately dropped. The piece also never says what the 1,700 PRs actually *were*, which matters: a factory merging seventeen hundred trivial dependency bumps tells a very different story than one merging seventeen hundred features.

What is left out: security incidents, the cost of running the validation layer, what happened when seats disagreed with the human rather than with each other, and any comparison against a non-factory baseline. The author's honesty about the honeymoon phase partially redeems this — but a retirement memoir that declines to publish its own numbers is still curating its legacy.

## Related

- [[Harness Engineering in Practice]] — that page presents PAAA (Purpose, Articles, Actors, Artifacts) as a framework arriving second-hand; this source is the first-person origin story for it, strengthening the page's governance thesis with the six months of incidents that produced each component.
- [[How to Build an AI Software Factory]] — Firecrawl's five-stage control plane leads with architecture and economics; this piece complicates it by arguing the converged architecture is the least interesting layer, and that the stages Firecrawl productizes are exactly the ones no platform ships.
- [[Harness Engineering is not Enough]] — the article explicitly positions itself as a retirement memoir alongside Dex Horthy's six-month return to reading all the code; this source strengthens that piece's skepticism with an independent factory run hitting the same wall from the inside.
- [[Building Autonomous Goal Loops That Deliver]] — that page's thesis that the harness must preserve lessons across sessions is borne out here at organizational scale, with the author's decision-records-as-side-effect practice as a concrete mechanism the goal-loop design lacks.

---
*Sources: [[raw/what-matters-after-the-software-factory-works]], [[summary/what-matters-after-the-software-factory-works]]*
