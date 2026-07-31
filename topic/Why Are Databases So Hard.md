# Why Are Databases So Hard

A database reliability engineer's clear-eyed walk through why databases are a perennial source of production outages — not because engineers are incompetent, but because the speed of light imposes a trilemma between correctness, performance, and availability that no architecture can escape. The article traces the problem from a single happy instance through HA to geographic distribution, showing that each step toward reliability adds time, and that time compounds until you hit a hard physics ceiling with less than one order of magnitude of headroom left.

---

## Key Quotes

> All practical implementations have to balance the opposing concerns of correctness vs. performance & availability (this is kind of similar to CAP theorem, but not exactly the same). Perfect correctness with no data loss across geographic distances would result in a database which is too slow or too costly to be useful for most applications. And these constraints cannot be overcome because it's the physical bounds of reality which imposes these limits.

This is the thesis, and it's sharper than CAP because it names the mechanism: time. Not a theoretical tradeoff between three abstract properties, but a concrete physical constraint. Every byte of data takes time to move, and that time compounds with every copy you maintain.

> If you were able to send data at the speed of light, it would take 16 milliseconds! And that's a one-way trip. To let our primary database receive a confirmation that the data was received we need a minimum of 32 ms. [...] A real network request therefore takes more like 66ms per trip, and a 130ms round-trip time. I have yet to experience any commercial enterprise willing to accept database write latency of 130ms.

The gut-punch numbers. NY→LA at the speed of light is 16ms one-way, 32ms round-trip — the theoretical *minimum* physics allows. Real networks are ~4× worse. And there's less than 4× improvement theoretically possible. This is the ceiling, forever.

> There is not even a single order of magnitude left between our current performance and the maximum allowed by the laws of physics. We cannot optimize time much more than we already have! No matter how advanced technology of the future becomes, this same problem will still exist until the end of the universe.

This is the article's most important claim: we're near the physical limit. There's no 10× breakthrough coming. Better fiber, better routing, better protocols — none of it cracks an order of magnitude. The speed of light is the speed of light.

> Most companies use a strategy of creating an HA cluster with strong consistency guarantees within a single datacenter only, and then using an "eventual consistency" approach to shipping data to another geographic location.

The industry's pragmatic settlement. Strong consistency inside the DC, async replication across DCs, and a frank admission during disaster recovery that some data loss is expected. Not elegant, but honest.

> I will talk with an engineering team one week that stresses how important consistency guarantees are for them. Sure, I can do that. Then the next week I'll talk to another team that demands the fastest performance possible. [...] Then after an incident I'll be yelled at by a manager who says we need to make availability our highest priority because our largest customer is threatening to churn. Then as we approach the end of our fiscal year I'll have other people breathing down my neck saying we need to cut costs.

The organizational dimension. The physics problem is immortal, but the organizational problem — different stakeholders cycling through conflicting demands on a fixed constraint surface — is what makes the job Sisyphean. You can't satisfy all four demands simultaneously, and the priority order changes by the week.

## Key Themes

- #database #distributed-systems #consistency #CAP-theorem #reliability #SRE #tradeoffs #physics

## Critical Analysis

**What this gets right.** The article's physicalist framing is correct and underappreciated. Most database discourse treats consistency/availability tradeoffs as implementation details — use synchronous replication if you care about correctness, async if you care about speed. gtowey correctly identifies that this isn't an engineering choice; it's a physics constraint. The NY→LA numbers make it visceral: 32ms is the *floor*, not the ceiling. There is no engineering workaround for the speed of light.

The step-by-step structure (single instance → HA → geo-distribution) is pedagogically brilliant. Each step introduces exactly one new constraint, and the cumulative weight lands with real force by the time you reach Step 3. This should be required reading for every engineer who has ever blamed "the database" for an outage without understanding what they're asking the database to do.

**Where it's incomplete.** The article names the cost/dimension the tradeoff adds (3×–5× data copies) but doesn't explore it quantitatively. What does a 5× storage multiplier actually cost at scale? The answer varies enormously by workload and would strengthen the argument.

It also gestures at the "bad query DOSes your database" problem in the closing paragraph but doesn't develop it. That's a different class of failure — not a physics problem but a resource isolation problem — and it deserves its own article. The two get conflated in production postmortems all the time.

The multi-database prescription (Redis + MySQL/PostgreSQL + KV stores) is presented as the practical resolution but without any guidance on *how* to make those choices. "Make sure engineers are choosing the right location for their data" is the whole job; the article ends right where the hard part begins.

**What's missing from the conversation.** The article doesn't mention consensus algorithms (Paxos, Raft) by name, which is a curious omission given that they're the standard answer to "how do we keep N copies in sync." Perhaps intentionally — the point is that even with perfect consensus, you still pay the network round-trip, and consensus adds its own latency on top.

More importantly, the article doesn't address the counter-argument from disaggregated architectures like [[Aurora DSQL]]: if you decouple compute from storage and push consensus into a purpose-built coordination layer, can you hide enough of the latency to make geo-distributed strong consistency practical? The answer appears to be "sometimes, with synchronized clocks and optimistic concurrency" — but the article's physics argument suggests the fundamental limit still applies.

**The bottom line.** This is the clearest single-page explanation of why databases are hard that I've read. It should be bookmarked by every engineer who touches production infrastructure. The physics argument is airtight; the organizational coda is painfully honest. The multi-database prescription is directionally correct but underspecified — which is fitting, because the article's entire point is that there is no general solution.

---
*Sources: [[raw/why-are-databases-so-hard]]*
*Last updated: 2026-08-01*
