# Harness Engineering is not Enough

Dex Horthy's AI Engineer talk diagnosing why "lights-off" software factories fail: the root cause is a model training problem (RL can't reward maintainability), not a harness-engineering gap—and the practical path forward is four-stage AI-assisted upfront planning so humans can still read every line without drowning in slop.

---

## The Diagnosis

Horthy opens with data points that have become ambient anxiety for teams shipping AI-generated code: outages attributed to coding agents, PR review quality plummeting (more comments, longer comments, more unreviewed merges), incidents and bugs per developer rising. The prevailing narrative says this is a skill issue—you're holding it wrong, you need more tokens, more loops, more adversarial review agents. Horthy's thesis is that **this is not a skill issue; it's a model training issue.**

The argument hinges on how coding models are trained. Current benchmarks like SWE-bench reward binary correctness: did the tests pass? There is no reward signal for maintainability because the cost function of bad architecture is measured in months and years—far outside any feasible RL horizon. The result: models learn to hack tests green with patterns that erode structure (pointless try-catch blocks, unnecessary type casts). They optimize for the test, not the codebase.

> "If you can't verify the maintainability of the code, it gets way harder to train on this stuff. … The cost function of bad architecture is measured in months and years."

This is Horthy's core technical claim, and it's a strong one. He's not arguing that models can't write clean code when explicitly prompted—they can. He's arguing that the *training objective* actively selects against maintainability because maintainability is too slow a signal to propagate through RL. This is a real constraint, not a temporary one, and it means that **no amount of harness engineering—adversarial review agents, loop maxing, prompt refinement—can compensate for what wasn't reinforced during training.**

> "If the model knew what good code looks like, it would probably write it in the first place."

The claim has teeth. Review agents and post-hoc verification can raise the floor, but they're constrained by the same model limitations. You can't review your way to maintainability any more than you can test your way to quality—the architecture is either there or it isn't.

## The Bitter Lesson Tension

Horthy acknowledges the obvious counterargument: the bitter lesson says scaling compute beats clever engineering, so maybe GPT-7 just solves maintainability. His response—"bitter lesson be damned, we've got some problems to solve"—is honest but unsatisfying. It's a pragmatic punt: yes, maybe brute force wins eventually, but codebases are degrading *now* and we need practical answers *now*.

The talk doesn't engage deeply with the possibility that better verifiers (which HumanLayer is building) could close the RL gap. If you can build a verifier that scores maintainability, you can train against it. This is the obvious synthesis Horthy gestures at but doesn't develop.

## The Practical Response: Planning Upstream

If you can't eliminate human code review, the leverage is to make review fast and pleasant. Horthy's answer: **30 minutes of AI-assisted upfront planning saves hours of review.**

His four-stage workflow:

1. **Product review** — What problem are we solving? Desired behavior, mockups. The human clarifies intent.
2. **Architecture** — System-level design: component contracts, data models, constraints. AI helps draft, human approves.
3. **Program design** — Low-level layout: types, method signatures, call stacks, call graphs. This is the stage Horthy thinks is most under-emphasized in agentic coding—people assume architecture is enough and the agent can fill in the rest. It can't, or at least not without degrading structure. Dylan Mulroy at Cloudflare uses call-graph-based planning here.
4. **Vertical slices** — Implementation order, multi-repo coordination, and validation checkpoints per slice. The goal is to reduce the chance of rework so that reviewing every line remains feasible and fast.

> "If you're drowning in PRs, you actually have too many bad PRs. Because a good PR is a joy to review. You're just reading through like, yep, this is great. This is what we discussed."

This reframe is powerful: the problem isn't volume, it's quality. A PR that matches an agreed-upon plan is trivial to review—you're verifying intent, not discovering it.

## What's Missing

The talk is a crisp diagnosis and a coherent prescription, but it leaves big gaps:

- **No threshold definition.** Horthy says agents struggle after 3–6 months, but provides no metrics, warning signs, or heuristics for when a codebase crosses the line.
- **No scaling model.** The four-stage workflow is described for a single feature with a small team. How it scales to dozens of concurrent features, cross-team coordination, or organizations where the architect isn't the implementer is unexplored.
- **No bootstrapping path.** How do you apply this to a codebase that's already degraded? The planning workflow assumes enough architectural clarity to plan against; brownfield codebases often lack it.
- **No economic quantification.** "30 minutes saves hours" is asserted, not measured. How often does planning prevent rework versus over-engineering? What's the false-positive rate?
- **The verifier gap is the real prize.** The talk's deepest insight—that maintainability can't be trained because it can't be verified at RL timescales—points directly at a solution Horthy only teases: build better verifiers. If you *can* measure maintainability quickly, you can train against it. This is where the talk gestures at HumanLayer's roadmap but doesn't deliver.

## Where This Fits

Horthy is charting a middle path between two camps: the "lights-off" maximalists ([[Cloud Software Factories]], [[StrongDM Factory Techniques]], [[The Dark Factory is a DOT File]]) who treat code as opaque weights validated by harness not review, and the "slow down" critics ([[The AI Productivity Paradox]], [[Five Studies That Are Changing How I Think About AI in Software Engineering]]) who document the downstream breaks without offering a constructive path forward.

His planning workflow rhymes with [[SDDW (Spec-Driven Development Workflow)]] and [[SDD Case Study — 13 Apps in 70 Days]]—the insight that planning upstream makes implementation cheap is converging across practitioners. It also echoes [[The Plan Is the Program]] and [[Vibe Coding as a Team Sport]] in treating the plan as the durable artifact and the code as its verification.

The maintainability-as-training-problem argument is the talk's distinctive contribution. It's the clearest articulation yet of why [[Loop Engineering]] alone isn't sufficient, and why [[Agentic Code Review]] (even at Cloudflare's scale with [[Orchestrating AI Code Review at Scale]]) can't compensate for what wasn't learned. It pairs naturally with [[Code Cleanliness and Coding Agents]]—SonarSource's finding that cleaner code cuts token consumption and file revisitations—and with [[Constraint Decay]]'s finding that structural constraints are the primary failure driver for coding agents.

The talk is also a useful counterpoint to [[The End of Code Review]]: Monperrus argues human code review is becoming indefensible; Horthy argues it's becoming *more* essential, but only if you shift the leverage upstream so review is lightweight verification rather than painful discovery.

---

*Sources: [[raw/why-software-factories-fail-dex-horthy]] (canonical), [[raw/harness-engineering-not-enough-dex-horthy]]*
*Last updated: 2026-08-01*
