# Two Languages — Complexity Bounds Systems Thinking

A ytx-gist transcript (digest + full transcript) of a ~45-minute conference panel — very likely Craft 2026, since it references Dave Snowden keynoting the previous day on "constructors and scaffolding" and a "Simon" keynote that morning. Nigel Thurlow (ex-Toyota) moderates Snowden, Diana Montalion, and Daniel North through one big claim — complexity science did to systems thinking what quantum mechanics did to Newton: bounded it, invalidated parts of it — and its consequences for root cause analysis, Agile, Ashby's Law, and AI. The diagnostic half is sharp; the prescriptive half collapses, and the gist's own digest catalogs the wreckage honestly.

---

## The argument

**The succession claim.** Snowden: the clue is "in the two languages — systems thinking and complexity science." Complexity did not refute systems thinking; it bounded it. The genealogy he sketches: general systems theory plus cybernetics, with cybernetics splitting into Ashby (data-signal information — "the foundation of modern systems thinking") and Bateson (abduction and metaphor, the lineage Snowden works in with Nora Bateson). System dynamics (Forrester → Meadows → Senge) is the popular branch and his main target: too few feedback loops to model a complex adaptive system, and archetypes that harden into categories people get forced into. Soft systems (Checkland, Snowden's own origin) works unconstrained but is facilitation-bound, so it cannot scale. Complexity itself splits: Santa Fe computational complexity is "abstractions without context" (his US Navy debate opponent claimed "if you give me enough money, I can build a model which represents the universe"); anthro-complexity instead stimulates the system, manages only actants and interactions, and shifts energy gradients as patterns stabilize.

**Cynefin's real lesson.** North's "massive epiphany": the domains are "aspects of a thing rather than categories you put a thing into" — his long-running pronunciation joke makes the point ("Ken Evan is not a quadrant model"). Any non-trivial system of work has emergent and ordered elements simultaneously. Snowden adds the inversion most complexity people miss: Cynefin is unpopular with purists *because* it insists some things genuinely are ordered — humans create structure, and you should use it where it exists.

**Root cause is domain-dependent.** The panel's most practical stretch. In ordered systems (manufacturing, Goldratt's theory of constraints) root causes exist. In complex systems there is "no linear material causality," and RCA there produces *retrospective coherence* — a tidy story that makes the system worse, not better. North's substitutes: ask what in the structure of the system of work made the failure *inevitable*, and what unintended consequences a fix would introduce. His Google case study — mudslide, fire, and a severed undersea cable hitting data centers simultaneously — ends with the root-cause verdict "We'll take those odds": preventing recurrence would slow everything else down. Montalion's substitute: "The call is coming from inside the house" — the problem is usually inherent in what you value and are certain about, so check how you are reinforcing it by trying to solve it.

**Agile's two heresies.** (a) The "empirical heresy": for Agilists, empiricism means "what I did on my last three projects" — or, in the SAFe variant, "things I half remember from four projects." Real empiricism is hypothesis validation. (b) Reductionism: decomposing "to the lowest level of coherent granularity" is fine and necessary; the error is "assuming that the whole can be derived from the parts." Hence the verdict: "Agile went wrong because it used the language of complexity, but it didn't use the practice of complexity."

**Ashby is not gravity.** The Law of Requisite Variety rests on Shannon's restricted data-signal view of information; human information flow is richer (pheromones as trust signal; constructor and assembly theory's claim that information has momentum, linking to new materialism). And it can be reversed: post-9/11, Snowden's counterterrorism work had symmetric governments use citizens as sensor networks to outprocess asymmetric threats. North's counter is the healthiest sentence in the session: "My counter to Ashby would be people are messy."

**Individual vs. interactions.** Snowden's stated disagreement with Montalion: both traditions focus on interactions — "actants, not actors" — and treat individual-focus as culturally specific to North America and Northern Europe. Montalion's defense: she writes for individuals because their thinking is what blocks complexity engagement — "the starter way." Nobody reconciles the two levels.

**Language solidifies into what you abhorred.** Montalion, via Huston Smith: "into every religion, when the prophet dies, into it comes everything they abhor" — insights in imperfect language become "solidified into exactly what you didn't mean." Historical proof offered: the Constantine moment, and the Jesuits' near-miss synthesizing Catholicism and Confucianism in China. Snowden is already fleeing his own label: "complexity science" now means agent-based modeling to people.

**AI: complication, not complexity.** Montalion first ("I would call it complication"), Snowden concurring — "and it's making them dependent on algorithms." The claimed gap: humans reason abductively, AI inductively, and "everybody is writing software based on inductive reasoning." North: LLMs are "a really, really useful translation tool at scale"; the money flooding them starves the rest of AI. His chess evidence: grandmasters under MRI are "remembering a game that has never existed" — recall into a smaller solution space, not faster scanning.

## Key quotes

> "Newtonian physics is not invalidated by quantum mechanics, but it's bounded." — Dave Snowden

The whole thesis in one line: succession, not refutation. It is also a self-protective frame — "bounded" lets him keep whatever parts of systems thinking he likes (Checkland, Ackoff) while condemning the popular branch.

> "Agile went wrong because it used the language of complexity, but it didn't use the practice of complexity." — Dave Snowden

The panel's most quotable sentence and its most defensible: the Agile community adopted complexity's vocabulary (emergence, adaptation) while running planning, estimation, and governance on ordered-system assumptions.

> "The problem is assuming that the whole can be derived from the parts. That's what reductionism is." — Dave Snowden

Careful and correct: he is *not* anti-decomposition. "Lowest level of coherent granularity" is the salvageable core — a useful discipline for anyone breaking agent systems into tasks.

> "Looking for root causes is a complete and utter waste of everybody's time." — Dave Snowden, in complex systems

The scoping matters: in a constrained, ordered manufacturing system, root causes exist. The failure is category error, not the tool.

> "We'll take those odds." — Daniel North, recounting Google's post-incident verdict

The best engineering story of the panel: mudslide, fire, and a severed undersea cable hit simultaneously; the checks and balances needed to prevent recurrence would slow everything else, so Google accepted the risk. Risk-acceptance as the honest output of analysis.

> "My counter to Ashby would be people are messy." — Daniel North

One line that does more to humanize requisite variety than Snowden's entire information-theory critique — and it is aimed at both speakers at once.

> "I'm not sure that I would call it complexity. I think I would call it complication." — Diana Montalion; Dave Snowden: "You actually stole my language. I wrote down AI isn't making things complex, it's making them complicated."

Convergent, independently reached, and the panel's only moment of genuine agreement — though it immediately dissolves into Snowden recruiting participants for his abductive-capability program.

> "It's not sometimes hallucinating, it's always hallucinating. And sometimes those hallucinations are useful, but it's bullshitting." — Daniel North

His full position is sharper than the soundbite: "hallucinating suggests some kind of cognition." LLMs are translation tools at scale being shoved into a "write my code" scenario they were not designed for.

> "Systems thinking in the main is focused on design without emergence." — Dave Snowden; Diana Montalion: "I definitely do not mean design without emergence."

The jab that draws blood on stage — and, notably, misfires: Montalion's whole architecture practice (iceberg model, leverage points, mindset change) is precisely an emergence-adjacent account. The correction goes unprocessed.

> "It makes things simple. It doesn't make systems thinking simple… it makes systems thinking simplistic." — Dave Snowden, on Montalion's book; her riposte: the publishers "did sell more books."

The panel's nastiest exchange, dispatched in two sentences. Her riposte is better than it sounds — it names the incentive (publishable simplicity) that guarantees the solidification problem she and Snowden otherwise agree on.

> "Human systems are messily coherent." — Dave Snowden

His 20-year-old coinage, "stolen and not attributed many times." The coherence matters, but so does the mess — a warning against every framework that resolves the mess.

## Key themes

- **#concept — Succession, not refutation.** Complexity science bounds systems thinking the way quantum bounded Newton. The map matters more than the slogan: system dynamics is the soft target; soft systems fails on scalability; the Ashby/Bateson split inside cybernetics is the deepest fault line.
- **#pattern — Cynefin as attractors, not boxes.** Domains are aspects overlaying a situation, not filing categories; non-trivial systems of work mix ordered and emergent elements; and insisting that *some* things are ordered is Cynefin's actual heresy against complexity purism.
- **#concept — Domain-dependent causality.** RCA is an ordered-domain tool; in complex systems it manufactures retrospective coherence. The substitutes on offer: structure-made-it-inevitable plus unintended-consequences (North), self-reinforcement audit (Montalion).
- **#tool — Anthro-complexity's method.** Stimulate the system → manage actants and interactions → shift energy gradients as patterns stabilize; distributed micro feedback loops instead of Meadows-style leverage points; micro-narrative with self-interpretation at scale (his "epistemic justice" work, never defined here).
- **#concept — Ashby as constraint, not law.** Requisite variety rests on Shannon's restricted information theory and is reversible via distributed sensing — treat it as a design constraint, not gravity.
- **#concept — Complication ≠ complexity; inductive ≠ abductive.** AI adds complication and inductive reasoning; the unanswered question is what software that enables abductive reasoning would even look like.

## An opinionated read

The diagnostic half of this panel is genuinely good intellectual history. The genealogy is real, the two-heresies indictment of Agile is precise and deserved — empiricism-as-anecdote is a live failure in every standup — and the domain-dependence of root cause is the most practically useful forty minutes of material here for anyone running postmortems on anything, including agent fleets. North's consumer stance is the session's epistemic asset: he keeps converting jargon into testable questions ("what in the structure of the system of work made this inevitable?") and his "people are messy" is the only sentence on stage that survives contact with an actual organization.

The prescriptive half collapses, and credit to the gist's digest for saying so in detail. The opening definitional question is never answered. North raises the mirror critique — "That's just a misapplication, though, surely — exactly the same thing happens in your quadrants" — and Snowden pivots away without answering it; Cynefin's own chronic misuse is exactly the solidification problem both speakers otherwise mourn. Montalion's "that's not what leverage points are, though" gets "that's a problem" and nothing else — Meadows is dismissed on vibes while her own book takes a drive-by insult. The moderator's only prepared question (advice to leaders on AI) is never delivered; the AI segment ends with Nigel's "AI is bullshit" summary, Montalion's "that's a binary" objection dying at lunch. And the panel's own prescription — write abductive software — is asserted with no method, no example, no architecture, just a participant-recruitment pitch. A panel about frameworks solidifying into what their founders abhorred that cannot say how *its own* framework escapes that fate is a panel that has demonstrated its thesis on itself.

Montalion gets steamrolled throughout, and that is the recording's real lesson about intellectual cultures: her interjections die of politeness, her one substantive disagreement (individuals vs. interactions) is announced as consensus rather than argued, and yet her defense — individuals' thinking is the blocker, so start there — is the most operational idea on stage. For this wiki's concerns the panel functions as an epistemics checksum: agent fleets are complex adaptive systems being managed with ordered-system tools (dashboards, root-cause postmortems, leverage-point reasoning, blanket "laws"). The panel's sharpest tools transfer directly; its sharpest omission — no test for telling ordered from complex in the moment — is exactly the judgment call agent operations makes hourly.

## Related pages

- [[Reducing Risk in Projects, Increasing Resilience]] — Snowden's Craft 2025 talk is this panel's earlier episode: the same anti-RCA thesis ("hindsight does not lead to foresight") and the same emergence-first method, here supplied with the genealogy and the Agile indictment that talk only gestured at. Read together, the panel strengthens the talk's core while exposing its weak spot — both records sell the method far harder than they test it.
- [[Architecture Is Designing Knowledge Flow]] — Montalion's Craft 2025 talk is the position Snowden jabs at on this panel ("design without emergence"), and the panel stages live the disagreement this wiki previously recorded as two separate talks. It complicates that note: Snowden mischaracterizes her even as she rejects the characterization, while her "call is coming from inside the house" answer here is the same mindset-first argument her talk makes better.
- [[Monte Carlo on Trial — The Probabilistic Forecasting Panel]] — the sibling panel transcript from the same circuit, with North and Thurlow trading moderator and combatant roles. Both records document frameworks failing public cross-examination, and the vocabulary failure that panel ended on ("we have to agree what we're arguing about") is the same failure this panel's digest catalogs — the pair makes a series about methodology debates that never reach shared definitions.
- [[Optimizing for Decision Points]] — that page builds agent-workflow guidance on Meadows' leverage points; Snowden's "I have huge respect for what Meadows wrote in the context of when she wrote it. But I have less respect now" complicates its premise directly. His substitute — many distributed micro feedback loops Meadows "could never have accounted for" — is arguably what that page's decision-point framing already practices, which suggests the critique lands on the vocabulary more than the method.

---
*Sources: [[raw/ac7d7c59e0742df5b628c3260e839ea8]], [[summary/ac7d7c59e0742df5b628c3260e839ea8]]*
*Last updated: 2026-09-13*
