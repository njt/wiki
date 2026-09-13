# Handling Complexity in Software Development Projects — Continental

Two Continental leaders — Balázs Lorant (economist, 25 years managing people, built the company's largest global AI development center in Budapest, heads AI for Autonomous Mobility) and Árpád Moroti (ADAS software architect) — use their automated-parking program to stage a deliberate trap for methodology thinking: automated parking looks waterfall-simple at the system level, but rated per architectural sub-problem on the Stacey Matrix it splits three ways (computer vision complex, path planning complicated, odometry simple), so the honest answer is not "agile or waterfall" but "which team gets which mode" — with architecture as the instrument that maps organization onto complexity, and the fine-tuning endgame admitted, in the speakers' own words, to be politics.

---

## The argument

**The trick.** At system level, automated parking is a Stacey-Matrix gift to waterfall: requirements "haven't changed in the last 50 years," the hardware is settled, "we've been using C since 20 years." Árpád then confesses the bait — "I have to admit that we have tricked you, and tricked you by intention" — because the rating that matters is per architectural sub-problem. A live Slido vote places computer vision as complex (can you even enumerate what to detect? AI or classical CV?), path planning as complicated, and odometry as simple ("simple as a stone"). The conclusion is organizational: teams are mapped onto the matrix the same way their sub-problems are, so one phase holds many work modes — agile and waterfall teams coexist and "it's not really a problem."

**Architecture as the connecting tool.** The bridge from complexity to organization is explicit: "You can use architectural design to identify complex problems... and organize your team according to that." Decompose the system (computer vision, path planning, odometry), rate each element, match a team to each element — the org chart becomes a projection of the matrix. This is Conway's law deployed deliberately rather than suffered.

**The lifecycle.** Three phases with different goals and modes. Pre-development: demo vehicle → OEM demos → feedback → rework → win the award; Scrum fits, colocation matters, and a change control board is the named anti-pattern ("You have to trust your teams"). Productization: automotive standards, target hardware, hundreds of contracted requirements; keep only tools and processes that "really help us," and keep documentation because you don't want to sit in a car built without quality management. Fine tuning: the base team in Europe ships to an application team colocated with the customer — Japan in the example — which tunes parameters, changes the UI, and files bugs while the base ships bug-fix releases. Innovation runs through all three, bottom-up and top-down: a new neural-network architecture with low-level fusion, and "radar vision parking" replacing ultrasonic sensors.

**Transitions are the danger zone.** Moving between phases you carry assets (know-how, confidence from beating competing suppliers, an empowered agile mindset) and liabilities (demo-phase technical debt, implementation gaps — "what happens if a camera is broken? Or there is snow... or the park markers are covered by leaves?", overconfidence, unknown unknowns) into a collision with strict automotive process requirements: documentation, architecture, requirements, traceability. Transitions must be consciously driven "otherwise you will end up with the chaos."

**Alignment is people, and the endgame is politics.** Cross-team synchronization in mixed-mode phases runs through "empowered interfaces" — specific trusted individuals — not standardized rituals, with "the most important job of a manager is to provide the context." In fine tuning, requirement uncertainty rises while technology locks down, and the talk says the quiet part loudly: "Aligning with your customer is always politics. Discussing the deadlines is politics." The prescriptions are correspondingly unglamorous: triage ("If you crash your car... you focus on the car crash"), a budget for flights ("buy those tickets to Japan"), and relentless communication of priorities.

## Key quotes

> "If you take the agile hammer and you put everybody in a scrum team with a scrum master, with the product owner and everything, this might not be the ideal way of doing it." — Árpád

The thesis in one sentence. From a 150-year-old supplier, after twenty years of agile-as-default: the scrum team is one tool among several, and the selection criterion is the sub-problem's quadrant — not fashion, not the org's certification portfolio.

> "I have to admit that we have tricked you, and tricked you by intention." — Árpád

The reveal that converts a boring talk ("parking is simple, do waterfall") into an interesting one. The pedagogical point is that system-level classification is the category error: the system belongs to no quadrant; its components each do.

> "If you want to do agile, really be agile... you won't probably be able to sell a bucket of requirements to the customer." — Árpád

Pre-development's economics: the sellable artifact is a working demo, not a specification document. The phase stays agile not because agile is virtuous but because nothing else closes the deal.

> "Maybe we don't need the standardized interactions and maybe we don't need a stand up each and every morning." — Balázs

The most radical sentence in the talk, stated almost casually: synchronization is achieved by "empowered interfaces," key colleagues who connect teams, and standardization is optional. In the Q&A Árpád doubles down — "You have to find the right individuals" — which is exactly where this model is most fragile.

> "The most important job of a manager is to provide the context." — Balázs

His summary principle for leading mixed work modes: with several work modes running at once, no single process can carry alignment, so the manager's product is shared context.

> "This is a political stuff, right? Aligning with your customer is always politics. Discussing the deadlines is politics." — Árpád

Fine tuning reframed: as deadlines near, requirement uncertainty rises while technology locks down, and the work becomes negotiation over schedule, content, and priorities. Most methodology talks pretend this phase away; this one budgets flights for it.

> "If your project is burning hot and your firefighters don't know what to do... this will come to a disaster." — Árpád

The cost of failing to communicate priorities in the endgame crisis. Triage — a car crash beats quality debts — only works if everyone knows what the fire is.

> "There are some people who don't like agile, there are some people who like agile and I sense that the drive is more from that direction... But yeah, to be honest, the rating of complexity is more or less done top down by the architects in this case." — Árpád, Q&A

The confession that undermines the talk's own mechanism: the classification that assigns work modes has no rubric, no reconciliation process, and is partly taste-driven. See below.

## Key themes

- #concept — **Complexity is per sub-problem, not per project.** The Stacey Matrix (requirement uncertainty × technological uncertainty) is applied per architectural component, and the placements differ: computer vision complex, path planning complicated, odometry simple. The system as a whole belongs to no quadrant.
- #pattern — **Mixed work modes within one phase.** Agile and waterfall teams coexist deliberately; a waterfall team can even depend on a Scrum team through timeboxing — milestones and deliverables at the boundaries, "freedom to act" (including make-or-buy decisions) inside the slot.
- #tool — **Architecture as an organizational instrument.** Decompose, rate each element, "organize your team according to that" — the org chart as a projection of the complexity map, backed by colocation that concentrates each topic in one site so task-force mode is one room away.
- #pattern — **Empowered interfaces over standardized process.** Alignment rides on trusted individuals, people rotation for know-how transfer ("easiest way: people transfer between teams"), and manager-provided context — not mandatory rituals. The model's bus factor is its known weakness.
- #concept — **Phase transitions as explicit asset/liability inventories.** Demo-phase debt, implementation gaps, overconfidence, and unknown unknowns are carried into productization's contractual, safety-process world; the transition must be consciously driven.
- #person — **Balázs Lorant** and **Árpád Moroti** (names as transcribed; the auto-transcript garbles both). The talk twice hand-votes the audience through **Kent Beck's** forest-or-desert ritual from the preceding talk, leaning on it as shared vocabulary.

## An opinionated read

This is the rare methodology talk from inside heavy industry that treats work-mode selection as an engineering allocation problem rather than a faith. The staged trick is genuinely good pedagogy: by first letting the audience price the whole system as simple, then forcing a per-component re-rating, it manufactures the insight instead of asserting it. And the honest parts are the best parts — the politics of the endgame, the triage, the flight budget, "your firefighters don't know what to do" — material most agile literature is too respectable to touch. Continental runs both universes in one company and thinks that's fine, which is a far more useful claim than either "always agile" or "always waterfall."

But the mechanism has a hole exactly where it matters. The rating is "more or less done top down by the architects," with no rubric and — by Árpád's own Q&A admission — partial preference contamination. If work modes follow taste as much as complexity, the elegant architecture-to-organization mapping risks becoming a post-hoc rationalization of org politics. Misclassification is never confronted either: odometry is "simple" until it isn't, and the cost of discovering a complex sub-problem under a waterfall team mid-project is precisely the scenario the Stacey framing exists to prevent. A talk about rating complexity needed a section on auditing the ratings.

The digest's sharpest catches stand. The talk invokes "very strict" automotive processes — documentation, traceability — as a hard contradiction with agile, then never names a standard (no ISO 26262, no ASPICE) and never explains how empowered scrum teams produce certifiable artifacts; the same silence surrounds the neural-network innovations, presented as wins with zero words on validating AI components in a safety-critical ADAS product. And the opening promise — "We made a lot of mistakes, failures, errors. Please learn from these" — is never cashed: not one concrete failure is dissected, so the promised war stories quietly become a brochure. Likewise the human cost of dragging an empowered agile team into documentation-heavy mode, and the perpetual maintenance bill of per-OEM clone-and-own forks, are named and dropped.

The relationship to Kent Beck's framing is the most interesting tension here. Beck insists forest and desert are parallel universes, not points on a spectrum, and that hybrids are unstable. Continental's answer is to vote desert-or-forest per sub-problem and run both universes simultaneously — which either refutes Beck's "not a matter of degree" claim or relocates it: the attractors may be real, but the unit of analysis is the component, not the organization. Meanwhile, Snowden's charge that Agile "used the language of complexity, but it didn't use the practice of complexity" finds a partial counterexample in this talk — a company actually varying practice by domain — yet the top-down, taste-tainted rating shows how much harder the practice is than the vocabulary, and the instrument (a two-axis matrix rated by architects) is cruder than anything the complexity crowd would endorse. The truth the talk demonstrates is less flattering than its thesis: organizations rarely get to classify complexity cleanly; they muddle through on architects' judgment, and the matrix is scaffolding for the conversation rather than a measurement.

## Related pages

- [[The Forest and the Desert Are Parallel Universes]] — complicates it. This talk hand-votes Beck's forest-or-desert ritual twice, then quietly refuses his binary by running both universes at once, one per sub-problem — evidence that the two attractors may be component-level facts rather than organization-level destinies, and that Beck's "not a matter of degree" claim depends on where you draw the system boundary.
- [[Two Languages — Complexity Bounds Systems Thinking]] — strengthens and tests it. Snowden's indictment of Agile meets a practitioner counterexample — work modes actually varied per domain — while his insistence that some things genuinely are ordered is exactly what "odometry: simple as a stone" asserts; the top-down, preference-tainted rating, though, shows how far the practice lags the vocabulary.
- [[Shaped by Demand — The Power of Fluid Teams]] — nuances it. Both talks treat org structure as a changeable variable ("the best structure... for now") with quarterly reassembly, but the triggers differ — North re-forms teams around visible demand, Continental around complexity jumps and contract phases — and Continental's contract-frozen base/app team split is a reminder that some structure is negotiated, not chosen.
- [[Architecture Is Designing Knowledge Flow]] — strengthens it from the industrial side. Montalion argues architecture is the flow of knowledge between minds; Árpád uses architecture as the instrument that maps teams onto complexity — two versions of "architecture is an organizational act," with this talk supplying the scale, the safety constraints, and the failure mode (alignment through irreplaceable people) that framing implies but never costs.

---
*Sources: [[raw/f599af9c05c7919a54bc0de408421f9f]], [[summary/f599af9c05c7919a54bc0de408421f9f]]*
*Last updated: 2026-09-13*
