# The Ten Properties of Software Quality

The Engineering Atlas guide's opening move is to reframe "is this software good?" into "which property is it missing?" — because *good* is not a property, while fast, correct, cheap to run, hard to break into, and easy to change are. It then walks ten of those properties (Correctness, Reliability, Performance, Scalability, Security, Maintainability, Operability, Usability, Cost, Compliance), each carrying its own decades-old body of practice. The fetched portion covers the framing plus the full first chapter, Correctness, whose thesis is that correctness can never be finished, only earned in layers — requirements that say what "right" means, designs that make wrong states unreachable, and tests that pin behaviour so it can't drift.

---

## Key Quotes

> "Stop asking whether a system is good and start asking which property it is missing. Good is not a property."

The guide's thesis in two sentences. It is a small reframe with a large payoff: "good" is a term of vague approval that ends a conversation, while "missing fast" or "missing correct" names a falsifiable target with an existing body of practice attached. It converts aesthetic judgment into engineering questions — the same instinct as [[Grug Brain Developer]]'s "no."

> "Dijkstra's old observation, the one behind *testing shows the presence of bugs, never their absence*, is not a counsel of despair. It is a design constraint."

Reclaiming Dijkstra from the fatalists. The common reading is resignation — "you can never prove it correct, so why bother." The guide reads it the opposite way: because you can't test your way to absence, you must design your way to it. That single pivot is what separates this correctness lane from a testing checklist.

> "The trap almost everyone falls into is trusting the green checkmark. A passing suite proves the behaviors you thought to check, and says nothing at all about the ones you did not. A suite can be 90 percent covered and nearly toothless."

Coverage as confidence, called out by name. This is the same false-confidence problem [[Test Validation and the Trustworthiness of Tests]] diagnoses from the other direction — and the guide's answer ("measure your tests with mutation testing") is a concrete, honest audit that the test-validation piece gestures at but never lands.

> "Two components that are each correct alone can produce garbage together when a message arrives twice, a replica lags, or two writers interleave."

The composition failure. Correctness is not compositional once concurrency and distribution enter — which is why the guide's most valuable cards are the ones that remove the possibility of the bug at design time (Make Illegal States Unrepresentable, Transactional Outbox) rather than hunt for its absence in test.

> "Every state you design out of existence is a whole family of tests you never need."

The design-first conclusion compressed to one line. It is the strongest claim in the chapter, and it is also where the guide's own caveat — "choose the guarantee your correctness actually requires, and no more" — bites: designing states out of existence is free only until it starts costing latency and availability you didn't need to spend.

## Key Themes

- **#concept Quality-as-missing-property**: Replace "is it good?" with "which property is missing?" Ten properties, each with its own practice lineage. The organisational spine that turns a flat catalogue into a walkable guide.

- **#pattern Design-out-the-bug**: The chapter's highest-value practice cluster — Make Illegal States Unrepresentable, Transactional Outbox, Consistency Models, Contract/Data Contracts. Prevention over detection.

- **#pattern Testing as the honest audit**: Testing Pyramid → Property-Based Testing → Mutation Testing. The suite is a safety net; mutation testing is the audit that tells you whether it actually bites.

- **#tool Mutation testing**: The one practice the guide tells you to adopt if you do nothing else — plant small deliberate bugs and check that something goes red.

## Critical Analysis

The reframe is genuinely good. "Good is not a property" is the kind of line that stays useful because it converts a vague word into a checklist of falsifiable questions, and it gives the ten-chapter walk a spine that a flat catalogue of best practices would lack. It lands near [[Software Engineering Craft]]'s territory — craft as judgment — but from the cataloguing direction of [[Software Engineering Practice Atlas]], of which this guide is the deep content behind the Engineering map.

The strongest move is treating Dijkstra's "testing shows the presence of bugs" as a design constraint rather than a lament. That reframing is what licenses the chapter's best advice: put the correctness work into types, contracts, and transaction boundaries so the bug has no representation to hide in. It is the same conclusion [[The Coming Need for Formal Specification]] reaches by a more formal route — the endpoint of "design out the bug" is "specify and prove," which is why the guide's practical middle (property-based testing, contracts, outbox) is arguably more useful to most teams than the formal endpoint.

The weak spot is the guide's own blind spot about trade-offs. It names ten properties but the fetched portion only acknowledges their mutual tension once, in a parenthetical ("every step toward stronger consistency costs latency and availability"). A reader who internalises "correctness is the property everything else stands on" and misses that caveat will over-invest in consistency and watch their p99 latency and cloud bill climb. The guide wants you to "choose the guarantee your correctness actually requires, and no more," but it never shows you how to make that choice — which is where the real judgment lives.

The honesty caveat from the parent atlas applies doubly here: this is AI-generated, unsigned content, and Correctness is chapter 1 of 10 — the showcase chapter, likely the most polished. The other nine chapters (Reliability through Compliance) survive in the fetched overview only as one-line slogans, and their depth is unverified. Treat the framing as a map and the Correctness lane as a well-lit trailhead; verify the rest before you rely on it.

## Cross-Links

- [[Software Engineering Practice Atlas]] — the parent site; this guide is the deep content behind its Engineering map
- [[Test Validation and the Trustworthiness of Tests]] — the same "green checkmark" false-confidence problem, from the test-validation direction
- [[The Coming Need for Formal Specification]] — where "design out the bug" ends if you push it far enough
- [[Software Engineering Craft]] — the hub this guide maps from the catalogue side

---

*Sources: [[raw/guides]], [[summary/guides]]*
*Last updated: 2026-08-21*
