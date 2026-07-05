# The Limits of Generalized Sync

Mikael Siidorow's master's thesis is the most rigorous empirical study yet of whether sync engines (Zero, ElectricSQL, PowerSync, Convex, etc.) can actually generalize across web applications. Short answer: **the read path generalizes; the write path doesn't.** Based on 69 practitioner sources, 13 interviews, and a production case study, the thesis classifies 14 sync engines into four architectural clusters, identifies seven trade-offs and five fundamental limits, and builds a decision framework for when to buy vs build. The online/offline boundary is the single most discriminating dimension — it determines everything downstream.

## Key Quotes

> "Sync engines form four architectural clusters rather than a smooth spectrum."

The "sync engine" label is misleading. You can't just pick one off the shelf — the choice space is lumpy, and picking the wrong cluster means fighting the architecture for the life of the project.

> "The online/offline boundary is the most discriminating dimension and sets the limit of generalization."

Offline doesn't just add a feature. It changes the authority model, the conflict resolution strategy, and whether the write path can be generalized at all. It's the architectural equivalent of a phase change.

> "Engineers adopt sync engines as state management replacements and API simplifiers rather than as synchronization tools."

This is the thesis's sharpest practitioner insight. The question isn't "do I need sync?" — it's "do I want to eliminate loading states and throw away my data-fetching layer?" Sync is a means; developer experience is the end.

> "ElectricSQL's architectural pivot from full active-active replication to read-only sync reflects a deliberate conclusion that the write path varies too much across applications to commoditize."

One vendor betting the company on this conclusion is a data point. Three vendors, multiple adopters, and two custom builders all converging on the same split is a finding.

> "Definitely the sync rules. Not only understanding them, but in some way it was hard to model actually how we wanted to request data." — B6, PowerSync adopter

Partial replication is the least-discussed dimension in practitioner content and the most painful in practice. This gap between what vendors talk about and what adopters suffer through is telling.

> "I would really try hard not to use PowerSync again… but I think I would have to again because there's nothing that replaces what we needed today." — B6

When offline is non-negotiable, the viable engine set collapses to essentially one option (PowerSync, for Postgres shops). That's not a market — it's a hostage situation.

## Key Themes

#distributed-systems #sync-engines #local-first #CRDT #web-applications #architecture-taxonomy #build-vs-buy #offline-first #research #case-study

## Architecture

The thesis classifies 14 sync engine instances across 13 projects along eight dimensions: offline capability, integration depth, authority model, partial replication, conflict resolution, read/write path architecture, sync unit, and network topology. The dimensions form a partial hierarchy — offline capability constrains authority model, which constrains conflict resolution, while the rest vary independently.

The engines fall into **four clusters**, not a spectrum:

1. **Database-pluggable** (Zero, ElectricSQL, PowerSync): Integrate with existing Postgres. Server-authoritative. This is where most existing applications land.
2. **Bundled platforms** (Convex, InstantDB, Jazz v2): Replace the backend. Greenfield-only. Fastest path to prototype.
3. **CRDT-decentralized** (Classic Jazz, Ditto, Automerge): Full local-first via CRDTs. Decentralized authority. The cost is constraining your data model to CRDT-compatible structures.
4. **Singular** (LiveStore, RxDB, Turso Sync, CouchDB, TinyBase): Each unique — no cluster.

The taxonomy is the thesis's main contribution: it's the first systematic classification of sync engines as integrated products rather than as implementations of CRDT theory.

## Critical Analysis

**The read/write asymmetry is the thesis's central insight, and it's underappreciated in the vendor ecosystem.** Every sync engine vendor markets the full stack — reads AND writes, local AND remote, online AND offline. But ElectricSQL bet the company on read-only, and the interview evidence shows adopters and custom builders independently converging on the same split. If the write path can't be generalized, then sync engines are really just very fancy cache invalidation — which is still valuable, but it's a different product category than what's being sold.

**The offline requirement is a trap.** It sounds like a nice-to-have ("wouldn't it be great if the app worked on the subway?"), but in practice it's a binary filter that eliminates almost every engine and forces you into a vendor relationship you may resent. Both PowerSync users independently confirmed it was their only option. That's not freedom of choice — it's architectural conscription. If you don't absolutely need offline, don't design for it.

**The partial replication gap is the elephant in the room.** It's the dimension with the lowest practitioner coverage (40 of 69 sources) and the highest operational pain. Sync engines are marketed as solving synchronization, but the hardest part — deciding which subset of data goes to which client — remains unsolved. Static rules are rigid, dynamic queries break on joins, and on-demand loading creates cascading latency. Every adopter built workarounds.

**The hidden costs finding is honest in a way vendor docs aren't.** Schema evolution across client versions, client-generated IDs, browser storage bugs, build toolchain integration, managed Postgres provider friction — none of this appears in the "get started in 5 minutes" tutorial, but every production adopter hits it. The case study's Render-specific hell (14-minute resyncs, custom WAL workaround, no event triggers) is a concrete warning: the abstraction leaks at the database layer, and your managed Postgres provider may not expose the features the sync engine needs.

**The build-vs-buy dynamic is inverted from what vendors assume.** Sync engines compete against custom implementations, not against each other. Multiple interview subjects said they'd build custom if they had to do it again. C1, C2, and C3 all chose to assemble components rather than adopt an engine. The question isn't "which engine?" — it's "engine or custom?" And for complex domains, custom still wins.

**The methodology is unusually rigorous for a field dominated by blog posts and conference talks.** 69 practitioner sources triangulated against 13 interviews, with explicit confirmation/extension/contradiction coding. The AI-assisted processing pipeline (Gemini 3.1 Pro extraction → manual timestamp validation) is a model for how to scale qualitative research on fast-moving technical topics. The single-coder limitation is acknowledged honestly rather than papered over.

**But the ecosystem is moving faster than the thesis.** Declared stable in March 2026 (Zero 1.0) and April 2026 (Jazz v2 public alpha) means some classifications are already stale. This isn't a weakness of the thesis — it's the nature of studying a field in active formation. The framework matters more than the specific classifications.

## Cross-Links

- [[Distributed Systems]] — The theoretical foundations: CAP, PACELC, consistency models that sync engines must navigate
- [[Databases and Data]] — Sync engines sit at the database-application boundary; the Postgres compatibility question is central
- [[Software Engineering Craft]] — The build-vs-buy decision framework, Brooks's essential vs accidental complexity
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The network fallacies that sync engines exist to hide
- [[Event-Driven vs Polling Architectures]] — Sync engines as an alternative to polling-based reactivity
- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for long-lived connections; sync engines face the same challenge
- [[SQLite is All You Need for Durable Workflows]] — SQLite on the client is the engine sync engines are built on
- [[Streambed]] — Postgres WAL streaming as infrastructure; sync engines use the same mechanism for different ends
- [[Building an AI Agent in Rails (Ionescu)]] — Production adoption of sync-like patterns in an existing application

---
*Source*: Siidorow, M. (2026). The Limits of Generalized Sync: A Taxonomy of Architectures, Trade-offs, and Decision Factors. Master's thesis, Aalto University. [aaltodoc.aalto.fi](https://aaltodoc.aalto.fi/server/api/core/bitstreams/d485ca46-ef01-41bc-ae4c-d468afb209a8/content)
*Fetched*: 2026-07-03
