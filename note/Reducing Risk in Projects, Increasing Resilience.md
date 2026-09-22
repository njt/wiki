# Reducing Risk in Projects, Increasing Resilience

Dave Snowden's Craft 2025 talk, transcribed by ytx gist tooling, applies complexity science to project risk. The argument in one line: you cannot predict in complex systems, so stop trying to close the gap to a target and instead manage the preconditions for emergence — map what the environment affords, distribute small decisions to natural groups, sense weak signals continuously, and fund what works rather than what people claim will work.

---

## The argument

Snowden opens by drawing a hard line: "complexity science is not systems thinking." Systems thinking says know where you must end up and close the gap; complexity says understand where you are and start journeys with a sense of direction — north stars are navigation aids, not destinations. In a complex system the only certainty is unintended consequences, so management means triggering emergence and monitoring patterns: give energy to good patterns, starve bad ones, and hold options open, because commitment to an outcome is itself risk.

Three empirical claims carry the talk. First, hindsight does not lead to foresight — with 10 dots yielding over 3.4 trillion possible combinations, "joining up the dots" is retrospective fiction, which is why root cause analysis and lessons-learned are broken. Second, you do not see what you do not expect to see: 83% of radiologists miss a gorilla 48× the size of a cancer nodule, and the 17% who see it conclude they were wrong after talking to the majority — so weak-signal detection needs distributed real-time human sensing, not experts and not periodic reports. Third, humans are terrible individual decision-makers but good small-group ones: we evolved to compromise in groups of seven or fewer and fall back into professional silos beyond that. The historical arc — scientific management, then the 80s engineering metaphor of BPR and Six Sigma ("engineers don't like uncertainty, so they tried to eliminate it"), now an ecological metaphor — ends with the talk's method in a sentence: manage the substrate, and see what works rather than deciding in advance what is right.

## Key quotes

> "There's no point in having a strategic plan to be the world's best cactus grower if you live in a paddy field."

The case for affordance mapping: strategy starts from what the environment permits, not from where you would like to end up. Most vision statements, he argues, land in the counterfactual zone — things that will not realistically change.

> "Hindsight does not lead to foresight."

The indictment of root cause analysis and lessons-learned in four words. The combinatorics argument is the substance: nobody could have joined the dots in real time, so post-mortems that ask "why didn't we notice?" are auditing the wrong faculty.

> "The 17% who do see it come to believe they were wrong when they talk with the 83% who didn't."

The radiologist-gorilla detail that makes weak-signal detection a group-dynamics problem, not an attention problem. Majority conferring actively destroys minority perception — the reason he wants non-experts in the sensing loop, not just experts.

> "I do not trust anybody who tells me to trust them."

From his stint as a deputy financial director. Distributed decision-making is built on auditability and the Panopticon effect — one anonymous member means everyone behaves as if observed — not on trust culture. A useful counterweight to the usual "psychological safety" framing.

> "The real money follows the people who've made things work, not the people who say they can make things work."

The funding inversion at the heart of the method: run 100 small auditable experiments, then fund what the ecosystem sustained, not what advocacy argued for.

> "If you don't know where the training data sets came from, you shouldn't be using the AI. It's that simple."

Delivered alongside the barb that AI "has been built by misogynist white males on the west coast of the United States who take Ayn Rand seriously after puberty." Training-data provenance as the first gate on tool adoption — the sharpest AI line in any Craft 2025 talk in this wiki.

> "One of the worst things anybody has ever said is to say Agile is a mindset. Total disaster."

Because it lets leadership blame a failed $5M SAFe rollout on workers' mindsets. His alternative: "very little of your decisions are made by your mind. Most are made by your body and your social environment" — hence micro-narratives and interventions over values posters.

> "Entangle yourself with your users."

The golden nugget, handed over via the MC's introduction — the whole talk compressed into one instruction for developers.

## Key themes

- #concept — Complexity science vs systems thinking; emergence; weak signals; hindsight's combinatorial limits
- #pattern — Distributed decision-making: natural groups of ≤7, four roles, one anonymous member, micro-budgets, fund-what-works
- #tool — Estuarine mapping (energy cost vs time to change), Hexi (method decomposition and recombination), microsensing capture
- #person — Dave Snowden, Cynefin's author, performing at full combative form: SAFe diagrams, Ayn Rand, and "six stigma" all take hits

## Opinionated take

The strongest material is the funding inversion and the micro-reporting of taboo signals. "Fund what works, not what people say will work" converts governance from advocacy-review into portfolio ecology, and it is actually auditable — a rare virtue in change management. The identity-stripped micro-report ("something doesn't feel right, not serious enough to report yet") is a genuinely clever answer to whistleblowing's too-late problem, and the Boeing metal fatigue example lands hard. Estuarine mapping is the most immediately usable artifact: a two-half-day exercise with an iron rule (disagree about what something is → break it down until you agree) that produces a shared map rather than a status update.

But the talk sells its own wares with little scepticism directed inward, and the gist's digest deserves credit for saying so. The new risk matrix is presented as "absolutely critical in any project environment" while being unpublished, unvalidated, and unaccompanied by a single worked example. The four decision-making roles are never named, which makes the whole distributed-decision apparatus unreproducible from the talk alone. The surveillance problem is waved away: continuous capture of engineers' work, instant polling of 2,000 employees, and anonymous observation are framed purely as burden-removal, with consent, GDPR, and Goodhart's law (micro-narratives get gamed once people know which patterns are watched) left unexamined. And the core tension goes unresolved — he demolishes lessons-learned for relying on hindsight, then builds anticipatory AI alerts by training on patterns from past failures, which by his own argument cannot see the novel failure modes that actually kill projects. Finally, "who judges which patterns get more energy?" is exactly the power question he forbids others from asking holistically; his methods quietly require someone with power to answer it.

Still, half the title is never defined: resilience is implied by distribution and sensing, never addressed as its own subject. The talk is best read as a methods catalogue with an epistemological sting — take the preconditions-not-outcomes stance and the napkin test, audit the rest.

## Relations

- [[Shaped by Demand — The Power of Fluid Teams]] — Dan North joked his Craft 2025 talk was "empirical, anecdotal sense... before Dave Snowden comes for me with pitchforks"; this is the pitchforks. Snowden supplies the theoretical underpinning North's demand-led planning lacked — natural groups of ≤7, Panopticon-anonymous audit, fund-what-works — and both talks share the enemy: big-framework adoption and SAFe.
- [[Architecture Is Designing Knowledge Flow]] — the direct counterpoint from the same conference. Montalion builds her architecture argument on systems thinking (iceberg model, Ackoff, Meadows); Snowden opens by declaring complexity science is *not* systems thinking and bans holistic thinkers outright. Read together, Craft 2025 hosted two opposing epistemologies of organisational change — and both cite the value of stories over abstractions.
- [[The AI Productivity Paradox]] — Cagan argues teams use AI to accelerate a broken project model; Snowden goes further and attacks the project model itself — kill reports, kill gap-closing, fund emergence. His training-data provenance rule also hardens Cagan's adoption scepticism into a checkable gate.
- [[Lean Software Production]] — Wynne says methodology is existential in the AI era; Snowden's Hexi is a concrete mechanism for that claim: decompose every method into its lowest coherent unit and recombine (a sprint peeled out of Scrum, replaced by a DSDM timebox) instead of adopting a single vendor's framework.

---
*Sources: [[raw/reducing-risk-in-projects-increasing-resilience-dave-snowden-craft-2025]], [[summary/reducing-risk-in-projects-increasing-resilience-dave-snowden-craft-2025]]*
*Last updated: 2026-09-13*
