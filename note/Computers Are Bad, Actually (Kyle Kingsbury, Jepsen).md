# Computers Are Bad, Actually (Kyle Kingsbury, Jepsen)

Kyle Kingsbury (Aphyr) — the Jepsen author who has spent 13 years getting paid to break databases — tells an Antithesis audience why distributed systems remain bad and how to test them anyway: fault injection, closed worlds, indefinite errors, explicit concurrency structure, and generative testing. The talk then pivots to the present tense: engineers shipping AI-generated code at three PRs a day without reading it are forfeiting the intuition and craft wisdom that programming itself builds, which makes rigorous correctness work more urgent, not less.

---

## Key Quotes

> "In short, databases were bad and they continue to be bad. But they're bad in different ways."

The thesis, delivered after a 13-year ledger: ~50 systems tested, 243 publicly discussable bugs — 63 lost updates and data loss, 35 ordering violations, 12 split-brain cases, 8 garbage reads ("you put in the number five and it would come back as a tuna fish sandwich"). The pessimism is measured, not performative: modern systems declare fault models and use real consensus. The bug-finding rate just never went to zero.

> "Any indefinite operation, any timeout, any unknown error is logically concurrent with everything else the system does for the rest of the system's life."

The sharpest single idea in the talk. Every distributed call has three outcomes — definitely okay, definitely failed, or unknown — and the unknown one stays live forever. A checker that reads "no leader available" as a definite failure will be wrong about half the systems it meets. Hence checkers that assert ranges: a counter is at least the acknowledged increments, at most the attempted ones.

> "We took a single-threaded program, and somehow we got concurrent execution out of it. And that should tell you something really uncomfortable about building distributed systems and testing them."

From two appends and one read you already get a 3×3 matrix of outcome combinations with one to five legal states; a third append makes 27 worlds. This is the argument for properties over enumeration: assert no lost committed data, no duplicates, no garbage, plus partial ordering constraints — and let the machine invent the inputs.

> "When I get a requirement like 'frozen accounts can't make transfers,' I need to ask: under what consistency model? And a lot of times that's not formally articulated in the design document."

Two histories with identical outcomes, differing only in timing: one is a bug, the other linearizability-legal. The requirement is meaningless until someone names the model — sequential, serializable, linearizable, session-scoped — and records the timing information a checker for that model needs.

> "Code is not the only outcome of the act of programming. … This is straight from Bainbridge's 1983 paper Ironies of Automation. When we don't do that work, we lose the context of the system itself."

The pivot to AI. Delegating the writing delegates the learning: the domain intuition, the feel for where abstractions lie to you, the *metis* (Detienne and Vernant's cunning craft wisdom) that balances underspecified, shifting requirements. The comic set piece — staring at a Claude PR asking "is there now a dependency on clocks in there? And is that a Ferris wheel? … is that fish tank load-bearing?" — lands because the point of AI-as-management is speed: "you might not have time to investigate them too deeply because this isn't just once every 6 months, this is three times a day."

## Themes

- #person Kyle Kingsbury (Aphyr), Jepsen; L. Bainbridge's *Ironies of Automation* (1983)
- #concept indefinite errors (ok / fail / unknown), closed-world testing, linearizability and consistency models, metis
- #tool Jepsen, Antithesis, FoundationDB's Flow simulator, Erlang's pulse scheduler
- #pattern fault injection / chaos engineering, property-based (generative) testing

## Analysis

The five tactics are not a checklist but a dependency chain, and the talk quietly makes that clear: fault injection supplies the failures worth testing; the closed world makes correctness properties checkable at all (an unseen writer once poisoned his X/Y invariant checker — "my assumptions were wrong, even though the test is telling me there's an error"); indefinite-error bookkeeping keeps the checker sound rather than flaky; recorded concurrency structure lets the checker ask whether *any* legal execution order explains the history; and generative testing is what you retreat to once enumeration explodes combinatorially. The Postgres anecdote is the emblem: Postgres's serializable isolation level had example-based tests for exactly the transactions its authors thought of, and Jepsen "almost immediately" found a violation in a three-transaction tree nobody thought to write down. Humans enumerate what they can imagine; randomness doesn't have that limit.

The AI conclusion is where the talk earns its title, and it is better than the usual slop lament because it is an argument about review economics, not aesthetics. If the organizing goal is speed, review becomes the bottleneck, and asking an LLM to review an LLM's PR is a loop with no ground truth in it. Kingsbury is scrupulous about his evidence — "I can't say if this is universal or not, I only have a few samples" — while reporting that even engineers confident in their own AI usage are "deeply frustrated" with the quality of what their systems produce overall. He also pre-empts the standard rebuttal ("we abstracted register allocation too"): maybe intuition is acquired at a higher level now. But the counter stands, because subtly incorrect software is precisely the failure mode you cannot review your way out of — and intuition is how you smell a load-bearing fish tank before production does.

Weaknesses worth naming: it is a tour, not a how-to; the tactics are expensive (fresh clusters, checkers that reason over execution orders), and the reassurance that you "can do them with a Perl script" is doing a lot of work in one sentence. The LLM-quality claim is anecdotal by his own admission. And the transcript is auto-generated captions, so quotes here are lightly cleaned approximations of his voice.

## Related Pages

- Strengthens [[Why Are Databases So Hard]]: that essay derives the correctness/performance/availability trilemma from physics; Kingsbury supplies the empirical ledger — 243 bugs across 13 years — showing the tradeoffs are being lost in shipped systems, not merely faced in theory.
- Strengthens [[AI Handles Incidents, Engineers Lose Touch with Their Systems]]: both lean on Bainbridge's *Ironies of Automation*; Kingsbury generalizes the skill-atrophy mechanism from incident response to programming itself and names what is lost — metis, not just system context.
- Strengthens [[The Coming Need for Formal Specification]]: the "complete breakfast" of designs, proofs, types, tests, and simulation is the same upstream-migration-of-rigor argument; Kingsbury adds simulation environments and chaos engineering to that list.
- Nuances [[State-Oriented Consistency]]: that piece says to ask which consistency each piece of state needs; Kingsbury's frozen-account example shows requirements documents rarely name any model at all — and his testing tactic makes naming one a precondition of testability.

---
*Sources: [[raw/watch]], [[summary/watch]]*
*Last updated: 2026-09-22*
