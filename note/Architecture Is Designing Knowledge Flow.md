# Architecture Is Designing Knowledge Flow

Diana Montalion's Craft 2025 talk (transcribed in a ytx gist, summary + full transcript) argues that architecture is not the arrangement of services but the design of how knowledge moves between people and technology. Software encodes its builders' mental models, so topology migrations fail unless the mindset that produced the old topology changes too; her toolkit is systems thinking — iceberg model, Beer Game, leverage points, capability reframing — aimed at the human system, not the diagram.

---

## The Argument

**Software encodes mental models.** Three months after Montalion's team migrated from monolith to microservices, they found the new services coupled exactly as their minds had been. The lesson generalizes: Pirsig's factory — tear down the system, leave the rationality standing, and "that rationality will just produce another factory." Transformation therefore means changing "the mindset out of which the systems, goals, power structures, rules and culture arise," all of which are tuned for a world of monthly releases.

**The iceberg explains why control fails.** A visible bug (event) sits on patterns and structures (test coverage, hiring) which sit on mental models ("run fast and hot is the way"). Organizations respond to the event by adding predictability and control — which reinforces the model that produced the event. This is one of two universal failure modes she says 50 years of systems research finds in *every* human system under complexity: **blame** (the Beer Game: retailers blame distributors, "product blames tech and tech blames product and everyone blames the architects") and **counterintuitiveness** (Forrester: "the people who know a system are often pushing the change in the wrong direction"; Brooks's Law is the canonical tech case).

**Relationships produce effect, purpose produces decisions.** Ackoff: a system "is never the sum of its parts, it's the product of their interaction" — eventual-consistency bugs and tuning-one-service-until-its-neighbor-dies are the software proof, and "if product hates tech and tech hates product, that architects the system" is the social one. Meadows supplies the epistemics: understanding the parts is "one," understanding the system requires understanding "and." Without a shared answer to "what's our fast package delivery?" (her DDD core-domain exercise for the executive team), engineers get asked to build a car boat.

**Knowledge flow beats knowledge stock.** Stock — whiteboard-testable expertise, "five years of JavaScript" — buys efficiency in your current context. Flow — "your ability to transfer knowledge between people and technology in ways that change and shift the system" — buys effectiveness, because with ~30% of any room on a different stack than a year ago, stock depreciates fast. Related discipline: trade opinion for knowledge ("here's how I can use what I know to help improve your ability to be effective," not "graphs don't scale").

**Capabilities and leverage points are the real work.** "Transformation doesn't scale through procedures and pipelines or CI/CD. It scales through capabilities. And capabilities are architected, not engineered." Reframe "implement the new payment system in six months" as "reduce cost per financial transaction by 20%" — the capability (processing transactions) is stable, the implementation churns. The leverage point is usually elsewhere: reconcile transactions across contexts rather than replace the payment system; make pipelines observable and self-healing rather than migrate CI; structure content for distribution rather than vomit MySQL through an API. Expect the 18-month rule: "you're too abstract, Diana" first, "yeah, we always knew that" later.

## Key Quotes

> "Our microservices are pretty tightly coupled, but our brains were very tightly coupled."

The talk's founding confession and its whole thesis in one line. The coupling was never in the diagram; the migration copied the org chart inside their heads into service boundaries. It's the cleanest first-hand demonstration that architecture work is cognitive work.

> "If a factory is torn down, but the rationality which produced it is left standing, then that rationality will just produce another factory." — Pirsig, the talk's spine.

She returns to this line three times, and it does real analytical work: it explains SAFe ("this is not what I mean today by transformation"), the car boat, and why the same organizations keep buying the same failure with new vocabulary.

> "You think because you understand one that you must understand two, because one and one make two. But you forget that you also have to understand 'and.'" — Donella Meadows.

Her favorite description of the job. The "and" is the space architects actually design in: eventual consistency, caching layers, one service tuned until its neighbor can't keep up. Every distributed-systems bug is an "and" bug.

> "If product hates tech and tech hates product, that architects the system."

The most radical sentence in the talk: interpersonal relationships are *load-bearing* structure. It quietly drops the pretense that Conway's Law is something that happens to you — the animosity is itself an architectural act.

> "Transformation doesn't scale through procedures and pipelines or CI/CD. It scales through capabilities. And capabilities are architected, not engineered." — flagged by her as "a little bit controversial."

The controversy is real and she's right to flag it: this is a direct claim that the tooling industry's whole transformation pitch (platforms, pipelines) is efficiency work wearing transformation's clothes. The reframe examples ("migrate Jenkins to CircleCI" → "reduce developer toil on pipeline maintenance by 50%") are immediately usable.

> "We need Kubernetes. Why do we need Kubernetes? Because we don't have Kubernetes." — Q&A diagnosis of platform fashion.

The purest statement of imitation-driven procurement in recent memory. To her credit she walks it back exactly far enough ("Kubernetes is a tool... it's fine") — the target isn't the tool, it's wanting a thing because others have it.

> "I'm a relationship therapist for technology."

Her job title when "systems architect" won't fit, and half a joke that isn't: if relationships between parts — human and software — produce the system's effects, then the architect's actual instrument is conversation, and the deliverable is changed minds.

## Key Themes

#concept #pattern #person

## Critical Analysis

The talk is diagnosis-heavy and construction-light. Its summary section honestly enumerates the gaps, and the biggest is that the centerpiece activity — finding leverage points — has no method attached. Asked how, she answers "you need experience... you got to put a system out in the wild and see what happens." That is a prerequisite, not a process: it cannot distinguish a real leverage point from a plausible wrong one before the 18 months are up, which is exactly the interval in which a disbelieved architect loses credibility, funding, or employment. Her answer to that survival problem is "that's okay" — an answer only available to someone with her seniority and book advances.

The "fire Steve" bit is the sharpest and least finished part. "Blame the system, not the people" and "so I say fire Steve" cannot both be load-bearing without a criterion for when a person *is* the system, and she offers none — just "there are lots of ways," none named. But the discomfort is honest: pure systems-thinking fatalism ("everyone is incentivized correctly") is how consultants avoid ever saying anything actionable, and the Steve joke is the moment the talk admits that some resistance is a choice. What's missing is power: mental-model change threatens whoever's status depends on the old model, and the talk never addresses what to do when leadership itself is the factory — the same leadership that "wants me to make a Gantt chart."

The most transferable idea, read in 2026, is the stock/flow distinction — and its conspicuous omission is AI. A 2025 talk about knowledge moving between people and technology that never mentions generative AI is either timeless or evasive. Read charitably, the framework anticipates the agentic shift precisely: knowledge stock is being externalized into models and retrieval, so the durable human value is flow — synthesizing across contexts, transferring judgment, helping others develop capacities. Montalion's own test cuts against AI enthusiasm too: an agent that answers with confident opinion rather than "experiences, information and learnings" fails her opinion-vs-knowledge bar on contact. But the talk doesn't make that argument, and its silence reads as a book trailer protecting its sequel.

Also skipped: the strongest counterargument to capability reframing, which is that "reduce cost per financial transaction by 20%" still requires someone to commit to a delivery, and outcome-language can be a way for architects to dodge accountability for dates. The talk treats "implement X in six months" as a factory request without engaging when it's simply the truth — deadlines and constraints exist, and pretending otherwise is its own rationality producing another factory.

## Related Pages

- [[Claude Is Not Your Architect]] — Montalion supplies the affirmative account Holland's polemic presupposes: if architecture were pattern selection, Claude could do it; because architecture is designing knowledge flow and changing mindsets, a pathologically agreeable model structurally cannot. Her "trade opinion for knowledge" test is exactly the standard Holland says AI fails.
- [[DDD Matters More When AI Writes Your Code]] — the "fast package delivery" exercise is a core-domain alignment ritual, and both sources land on the same claim: the scarce resource is shared understanding of purpose, not code or artifacts. Montalion's "no shared purpose, no decisions" is the organizational version of Smółka's "the value is that the team understands the domain."
- [[Nobody Knows How Large Software Projects Work]] — Goedecke's "systems exceed anyone's mental model" is the static diagnosis; Montalion's knowledge flow is the dynamic response to the same fact. Where he offers investigation as the substitute for recall, she argues the deeper fix is transfer between people — complicating his picture with the claim that stock in any single head is the wrong unit anyway.
- [[Engineering for Bounded Cognition]] — the iceberg's deepest layer is mental models, and bounded cognition explains why that layer moves so slowly: minds that hold four things can't re-derive their own assumptions on demand. Williams designs for the constrained mind; Montalion asks why the constraint is treated as a hiring problem rather than a design material.

---
*Sources: [[raw/architecture-is-designing-knowledge-flow-diana-montalion-craft-2025]], [[summary/architecture-is-designing-knowledge-flow-diana-montalion-craft-2025]]*
*Last updated: 2026-09-13*
