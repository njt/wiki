# State-Oriented Consistency

A practical design rule for distributed systems: consistency is not a property of the system — it is a property of individual pieces of state. Ask each piece what it actually needs, row by row, rather than defaulting to one consistency strategy for everything. The hardest part isn't choosing better algorithms; it's learning to ask a better question.

---

## The Core Insight

The Keel IoT team, building a clustered MQTT broker, kept asking "which consistency model should the cluster use?" — until an OOM kill incident forced them to realize that was the wrong question entirely. The right question: **"Which consistency guarantee does this specific piece of state actually need?"**

This reframe is the article's central contribution. It sounds small. It isn't. The first question assumes a single answer exists for the whole system. The second assumes it doesn't — and the second one is what the whole piece converges on.

## Uniform Consistency: Naming the Default

The authors name a design anti-pattern they'd fallen into:

> The default we'd fallen into deserves a name too, even though nobody chose it on purpose. We started referring to this design habit as **Uniform Consistency**: apply one consistency strategy to the whole system, and let every piece of state inherit it, regardless of what that specific state actually needs.

This isn't a specific algorithm — it's a *habit*. Reaching for whatever consistency model the system already trusts, everywhere, by default. Nobody designs a system by deciding "we will apply Uniform Consistency." It's what happens by default, one small decision at a time, when the question at the very start stays unasked.

The chain that leads from Uniform Consistency to production failure:

> Wrong abstraction → Wrong guarantee → Wrong architecture → Wrong scaling

And the authors' diagnosis: "An OOM is just where the chain happened to become visible. It could just as easily have surfaced as a slow memory leak, a scaling limit nobody could explain, or a rolling update that mysteriously got riskier as the fleet grew."

## The Classification Table

The article's centerpiece is a five-row table classifying different pieces of cluster state by their actual requirements:

| State | If two nodes briefly disagree... | Minimum sufficient guarantee |
|---|---|---|
| Live connection ownership | Duplicate delivery, or a ghost connection | Single authoritative owner, decided right now |
| Offline client ownership | Nothing bad — no live connection to duplicate | Deterministic — computable, no arbitration needed |
| Message routing | A briefly stale route, self-correcting | Available — tolerant of brief staleness |
| Durable message state | Silent, permanent data loss | Durable — no exceptions |
| Cluster membership | A node briefly looks alive when it isn't | Eventually consistent |

Five rows, five different minimum guarantees. The insight isn't any single row — it's that capable engineers will still default to Uniform Consistency unless something forces this question onto the table, row by row.

## The Framework

The generalizable five-step loop:

1. **Identify the state**
2. **Ask what happens if two nodes disagree**
3. **Find the minimum guarantee that's actually required**
4. **Choose the weakest mechanism that provides that guarantee**
5. **Repeat, for the next piece of state**

> The exact answers will differ by system. A scheduler, a database, a service mesh, and an MQTT broker won't classify the same state the same way — a scheduler's "who owns this task" isn't a broker's "who owns this session," and shouldn't be forced to be. The loop stays the same. Only the rows in the table change.

## Guarantees → Mechanisms

Each row in the classification table naturally suggests a different mechanism, because each row requires a different guarantee:

| State | Guarantee | Mechanism |
|---|---|---|
| Live sessions | Single authoritative owner | Coordinator |
| Offline sessions | Deterministic | Deterministic placement |
| Topic routing | Availability | AP routing |
| Durable message state | Durability | Durable shared storage |
| Membership | Eventual consistency | Membership gossip |

A key design rule emerged: **the coordination layer never carries bulk message traffic.** A coordinator that also carries payload caps your throughput at consensus latency.

## What Consensus Is (and Isn't) For

> People often ask which consensus algorithm we chose. That turned out to be the least interesting architectural decision we made. It's a solved problem — pick a mature implementation, and it does exactly the job a coordinator is for. The decision that actually mattered, and the one that's easy to skip past, was identifying which parts of the system deserved consensus *in the first place* — and, just as importantly, refusing to let that mechanism creep into the rows that didn't need it, just because it was already sitting there and already trusted.

This is the article's sharpest operational insight: consensus is a solved problem. The unsolved problem is **containment** — keeping consensus from colonizing every piece of state in the system just because it's the tool you have.

## The Result

The redesign removed an architectural assumption — that any node needed to be ready to serve any client — rather than optimizing within it. The memory floor that consumed most of the budget before serving a single real connection didn't get smaller through tuning. It disappeared.

> The architecture became simpler not because it had fewer mechanisms. It became simpler because every mechanism had exactly one job.

## Key Themes

#distributed-systems #consistency #consensus #pattern #architecture #coordination

## Critical Analysis

**This is a practitioner's article, not a research contribution — and that's exactly its value.** The authors are explicit: "We don't claim the idea is new. We found that giving it a name made it easier to reason about consistently." The contribution is naming and operationalizing a design habit (Uniform Consistency) and its antidote (State-Oriented Consistency), with a concrete incident-to-fix narrative that makes both memorable.

**The framework generalizes, but the table doesn't.** The five-step loop (identify, ask-what-happens-on-disagreement, find-minimum-guarantee, choose-weakest-mechanism, repeat) is genuinely portable across systems. But the specific rows in the table — live sessions, offline sessions, routing, durable messages, membership — are MQTT-broker-specific. The authors acknowledge this explicitly. What travels is the *reasoning process*, not the answers.

**The containment argument is underappreciated.** The article's most important claim isn't in the table — it's that once you've identified which row needs consensus, you must also actively prevent that mechanism from spreading to other rows. This is a social/organizational insight disguised as a technical one: teams trust tools they've already validated, and that trust becomes the vector for Uniform Consistency. The mechanism isn't the risk; the trust is.

**Where this fits in the distributed systems literature.** The article sits between the eight fallacies ([[21 Years and Counting of Eight Fallacies of Distributed Computing]]) and the SDPD failure-mode taxonomy ([[SDPD — Systems Design Police Department]]). The fallacies tell you *why* things break. SDPD tells you *what* the breakage looks like. State-Oriented Consistency tells you *how to design so fewer things break in the first place* — by asking each piece of state what it actually requires rather than defaulting to one answer for everything. It's a design-time complement to the diagnostic and operational tools those other resources provide.

**The article pairs strongly with Hebert's [[Queues Don't Fix Overload]].** Both diagnose the same class of error: reaching for a trusted mechanism (queues, strong consistency) without first identifying what the specific situation actually requires. Hebert's "red arrow" — the true bottleneck you must find before optimizing — maps directly to State-Oriented Consistency's "minimum sufficient guarantee" — the actual requirement you must identify before choosing a mechanism.

**What's missing.** The article doesn't address the organizational dynamics that make Uniform Consistency the default. Teams don't choose Uniform Consistency; they inherit it from the system's initial architecture and never revisit the question. The framework assumes technical willingness to classify state row-by-row but doesn't address the organizational inertia that makes such a review unlikely to happen without an incident forcing it. The article also doesn't explore how State-Oriented Consistency interacts with existing consistency-model taxonomies (CAP, PACELC, the consistency spectrum from strict to eventual) — it's a practical complement to those models rather than a theoretical refinement of them.

**Bottom line:** This is a sharply written, honest field report from the trenches. The OOM incident → investigation → reframe → classification → redesign arc is compelling because it's concrete. The article's real contribution is giving names to both the disease (Uniform Consistency) and the treatment (State-Oriented Consistency), making them discussable design choices rather than invisible defaults. Worth reading alongside [[Distributed Systems]]'s observation that agent orchestration keeps reinventing distributed systems primitives — this is the kind of hard-won distributed systems lesson that agent systems are currently repeating.

---

## Related Pages

- [[Distributed Systems]] — The hub page: agent orchestration IS distributed systems, and this article addresses the consensus-for-the-right-things gap
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — Uniform Consistency as an unlisted ninth fallacy: "one consistency model fits all"
- [[SDPD — Systems Design Police Department]] — Failure-mode taxonomy; State-Oriented Consistency as design-time prevention for many of those failure modes
- [[Queues Don't Fix Overload]] — The same class of error: reaching for a trusted mechanism before identifying the actual requirement

---

*Sources: [[raw/state-oriented-consistency-html]], [[summary/state-oriented-consistency-html]]*
*Last updated: 2026-08-07*
