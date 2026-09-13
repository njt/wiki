# Evolutionary Architecture — Maciej Jedrzejewski (Craft Budapest)

Maciej "MJ" Jedrzejewski's talk "Evolutionary Architecture: The What. The Why. The How." (ytx gist transcription, delivered at the Craft conference in Budapest; the transcript carries no year, though sibling talks in this wiki span Craft 2025–2026) prescribes an architecture lifecycle instead of an architecture decision: start as simple as possible, modularize from day one, and let observed architectural drivers — not vanity metrics — trigger each transition through Simplicity → Maintainability → Growth → Complexity. The disease it treats is the Project Paradox: the moment of maximum technical decision-making coincides with minimum domain knowledge. Growth is explicitly optional — "not all products will come to that point."

---

## Key quotes

> "Always choose an architecture based on your current needs and based on your current context. Not a wishful thinking."

The one sentence he'd keep if the audience kept only one — and the talk's whole argument compressed. Note the second half of the move, which most "start simple" advice omits: "don't close the door" — keep technical options open as long as possible. Simplicity with optionality, not simplicity as closure.

> "It's like you would hire the Formula One driver to prepare you a sandwich."

His cohesion heuristic: invoicing shouldn't schedule appointments. Everyone does the one thing they're extremely good at. The silliness is the point — it's a test you can run in a requirements workshop without a single architecture diagram.

> "Kafka, Kubernetes, Redis, microservices — they are great. But usually we are using them for our imaginary problems."

The anti-overcomplexity line, and careful in its fairness: the tech isn't wrong, the justification is. This is the talk's recurring split between components you own because your drivers demand them and components you own because your CV does ("let's do this, this, this and that... then on our CV that we worked with everything").

> "You might say what the hell, InMemoryQueue, but what if I lose this event? Well, you will need to fix it."

Scale-appropriate pragmatism for a karate-club-sized system: losing an occasional event at 1,500 patients is a repairable nuisance, not a design driver. The ladder exists precisely so you don't pay RabbitMQ prices for karate-club problems.

> "Yesterday I had one problem and I added cache to solve it. Now I have two problems."

Why cache is the *last* scaling lever: indexes and queries first, read replicas second, cache only when both are exhausted. The saying is old; the discipline of actually ordering the ladders is what most teams skip.

> "Our aggregate is a guardian of consistency... no business logic is happening around, outside of it, that can change it directly."

The chapter-four fix for the anemic model: the prescription entity was "stupid" only because several outside services mutated it. Event storming surfaces commands, actors, and policies; then mutation logic moves inside the object.

> "If you skip the strategic part, then you are hacked up."

His DDD verdict from a self-declared DDD evangelist: strategic DDD (subdomains → bounded contexts → context map) is for every business application, from day one; tactical DDD (aggregates, value objects, entities) is *not* for the start. Most DDD adoption fails by doing the second without the first.

> "If you put trucks into cars, then you will have a mess."

Bounded contexts are not permanent: the car-leasing company that buys a truck-leasing company needs new boundaries. Even the up-front strategic work carries an expiry date.

> "I was supposed to tell you a joke about evolutionary architecture by now, but it is still evolving, so I can't."

The talk's only joke and its thesis in one line — architecture as a process you're always inside, never an artifact you finish.

## Key themes

#person Maciej Jedrzejewski (MJ) · #concept Project Paradox, four-chapter evolutionary lifecycle · #pattern modularity from day one, schema-per-module, scaling ladder, use-case-per-folder code organization · #tool architecture tests ("solution structure tests"), event storming, Postgres-as-queue with inbox/outbox · #concept strategic vs tactical DDD

## The take

What makes this talk worth keeping is that it's *operational* where most simplicity preaching is not. "Start simple" is cheap; MJ instead specifies the folder layout (one project, folders mirroring cohesive areas, each use case owning its endpoint and rules), the database split (one database, one schema per module, cross-module access only through the owning module's public API), the messaging ladder, the extraction criteria, and even rescue tactics for systems that are already over-complicated (feature-by-feature decomposition preferred, tactical forking as the "quite of a hardcore strategy" fallback). The extraction story is unusually literal: extraction is "take the content of the folder, extract." Modularity from day one is framed exactly right — as cheap insurance that keeps future moves possible, not as premature structure. And the closing jab at CV-driven architecture names a real incentive the field rarely admits.

The gist's own digest is the most rigorous part of the source, and its holes are real. **No trigger metrics**: "architectural drivers" are invoked at every chapter transition but never defined — how much performance loss justifies extracting a microservice? Maintainability is the most-cited driver and the least measured. **No migration path**: every chapter shows an end-state diagram, never the transition — how do you split a shared database and move live medical data while 300 clinics keep running? The one audience question about mid-flight migration got punted as "opening a Pandora box." **Eventual consistency hand-waved in the riskiest possible domain**: modules "can duplicate the data on their side... whatever they want" is a lot of causal hand-waving for prescription data in healthcare, with no sagas, no compensation, no staleness story. **No observability**: odd for a methodology whose entire trigger mechanism is "we see that performance decreased." The 99% SLA ("four days downtime for the entire year") is stated in chapter one and then never used in any decision.

There is also a naming problem the digest flags and I'd underline: *evolutionary architecture* already has an established meaning in this field — fitness functions and incremental change as first-class concerns (Ford/Parsons/Kua/Sadalage). None of that appears. What MJ actually delivers is a **modular-monolith evolution narrative wearing a borrowed title**. Relatedly, the talk's central tension stays unresolved: "not wishful thinking" and "don't close the door" pull in opposite directions, because schema-per-module, contract extraction, and kept-open options are *themselves* speculative structure — insurance premiums paid before the accident. Where healthy optionality ends and speculative generality begins is exactly the line the talk never draws.

From this wiki's vantage, two things sharpen the talk. First, its "shift technical decisions as far into the future as possible" is [[YAGNI]] applied to architecture — and the modularity discipline is Fowler's boundary condition made concrete: effort to make the software easier to modify is not a YAGNI violation, it's what makes deferral safe. In an agent era this pays twice: use-case-per-folder modules are exactly the shape agents navigate well, and [[The Economic Benefit of Refactoring]] measured semantic decomposition cutting an agent's token bill by 83% — MJ's structure is that decomposition, prescribed from day one. Second, "leverage what you have" — Postgres-as-queue, inbox/outbox, read replicas before cache — is the same posture as [[Rust Scalable Backend Services]] and [[SQLite Is All You Need]]: the database you already run is the best new component, because it isn't new.

## Relate

- [[Designing Deploy-Time Flexibility for Modular Systems — Florin Coros, Craft 2025]] — the same-conference twin that strengthens MJ's thesis by dissolving the question MJ sequences: Coros makes deployment a post-compile decision via contract-only dependencies, while MJ defers it in time via extraction triggers ("teams stepping on each other's toes"); both lean on the same contract-extraction move, but Coros buys the option explicitly while MJ only keeps the door open.
- [[Essentials from a Real-World Microservices Journey — Sander Hoogendoorn (Craft 2025)]] — the direct counterpoint: Hoogendoorn runs 13 people across ~160 microservice repos while MJ stays in a single deployment unit at 10,000 patients and three teams; MJ supplies the monolith rebuttal that Hoogendoorn's own digest flagged as dodged.
- [[YAGNI]] — Fowler's four-cost economics is the theory behind MJ's practice: "not wishful thinking" is YAGNI for architecture, and schema-per-module plus contract extraction operationalize Fowler's "enabling practices" boundary — malleability, not presumptive features.
- [[DDD Matters More When AI Writes Your Code]] — complements: both argue DDD's value is strategic (cohesive areas, bounded contexts, ubiquitous language) rather than tactical ceremony; MJ adds the boundary-evolution warning ("trucks into cars") and the workshop pipeline (event storming, [[Domain Storytelling]]) that Smółka stops short of specifying.

---
*Sources: [[raw/178d71dd271ddba3a22bd71c5ce45cba]], [[summary/178d71dd271ddba3a22bd71c5ce45cba]]*
*Last updated: 2026-09-13*
