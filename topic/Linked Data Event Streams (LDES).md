# Linked Data Event Streams (LDES)

A SEMIC specification that defines how to publish append-only streams of RDF data over HTTP, using TREE hypermedia search trees for efficient client synchronization. Think Kafka for Linked Data — immutable members, traversable time-partitioned nodes, and a formal vocabulary for retention policies, version semantics, and transaction grouping.

---

## Key Quotes

> "A Linked Data Event Stream (LDES) is a collection of members that cannot be updated or removed once they are published, with each member being a set of RDF quads."

This is the atomic commitment. Not "should be immutable," not "preferably append-only" — *cannot* be updated or removed. LDES takes a hard stance: the event stream is a durable log, and everything downstream (versioning, retention, synchronization) is built on that guarantee. This is the same design move as Kafka's immutable segments or EventStoreDB's append-only streams, but applied to the Semantic Web stack.

> "The root node will contain all context information. The root node and any subsequent node will contain members and relations to other nodes."

A clean two-layer separation that mirrors the distinction between metadata and data in REST APIs. The root node tells you *what* the stream is and *how* to traverse it; subsequent nodes carry the payload. This is hypermedia done right — the root is a self-describing entry point that a client can dereference and navigate without out-of-band knowledge.

> "The client MUST ensure a member is only emitted once."

The specification's sharpest constraint — and the one that drives most of the complexity in state management. The spec notes that "keeping a list of all emitted members forever will become problematic for large LDES instances" and offers two escape hatches: unordered mode can drop members from state once their source page is confirmed immutable, and ordered ascending mode can use timestamp/sequence bookmarks instead of a complete member set. This is the kind of practical scalability concern that separates real specifications from academic exercises.

> "When no retention policy is provided in the root node, the consumer MUST assume that all members that have been added to the LDES are still available. When a retention policy is provided, however, a consumer MUST assume it will not be able to find members outside of the retention policy."

An honest contract between publisher and consumer. Retention policies aren't a warning — they're a boundary condition. The consumer's job isn't to guess what might be recoverable; it's to accept the publisher's stated limits and work within them. This is refreshingly direct for a domain (data publishing) that usually buries these guarantees in fine print.

> "In ordered mode, the client MUST ensure no other member can still be discovered that could precede the member that is to be emitted."

This is the hard problem in ordered synchronization: you can't emit a member until you've confirmed nothing earlier could arrive. The solution involves a priority queue over tree relations, logical AND combination of multiple relations to the same node, and the understanding that relations on non-supported paths (e.g., geospatial) must be prioritized immediately since following them might surface earlier members.

---

## Key Themes

- **#protocol** — A formal consumer specification with MUST/SHOULD/MAY semantics, defining synchronization as a deterministic algorithm over HTTP+TREE+RDF.
- **#pattern** — Append-only event stream as the foundational API. LDES inverts the typical data-publishing stack: instead of building query APIs with dumps as an afterthought, the stream IS the API.
- **#concept** — Immutable pages as a caching and state-management primitive. Nodes marked immutable can be fetched once and dropped from the client's frontier, solving the infinite-state-growth problem.
- **#pattern** — Version semantics embedded in the stream vocabulary. Create/update/delete objects plus version-of paths let consumers derive CRUD operations from append-only events — this is event sourcing for RDF.
- **#concept** — Retention policies as declarative subset declarations. Instead of the consumer guessing what's available, the publisher tells the consumer exactly what window of history to expect.
- **#tool** — LDES vocabulary and JSON-LD context providing terms for event streams, retention, versioning, and transactions.

---

## Critical Analysis

**The append-only bet is both the strength and the limitation.** LDES commits to a model where members are forever. This gives you replay, audit, and deterministic synchronization — exactly the guarantees that make Kafka and event sourcing valuable. But it also means the spec has nothing to say about member deletion, correction, or compaction. In practice, retention policies are the escape valve: a publisher can declare that old members are simply gone, and the consumer must accept that. The spec is honest about this, but it means an LDES is only "immutable" within the retention window — outside it, the guarantee evaporates.

**The specification is a consumer spec, not a producer spec.** This is an intentional scope decision, but it creates an asymmetry: the client's behavior is exhaustively specified (initialization, state management, HTTP handling, member extraction, tree traversal), while the server's behavior is left to a separate "server primer." This is the right call for a specification — you can't mandate how servers are built — but it means the hard problems of ordering, partitioning, and retention implementation live on the producer side with comparatively little guidance.

**The polling model is both pragmatic and dated.** LDES clients synchronize by polling (at `ldes:pollingInterval` or a client-chosen frequency). This is simple, stateless-on-the-server, and trivially cacheable — exactly the properties you want for a specification aiming to be "as lightweight and straightforward as possible to host." But it also means LDES inherits all the problems of polling: latency proportional to interval, wasted requests when nothing changes, and the N+1 problem for sources with many views. WebSub, SSE, or WebSocket-based push would give lower latency, but they'd also require persistent connections and server-side state that the spec explicitly avoids. The trade-off is clear and, for the target use case (open data publishing with thousands of consumers), probably correct.

**The RDF commitment is a double-edged sword.** Using RDF gives LDES everything the Semantic Web stack provides: standardized serializations, SHACL validation, SPARQL queryability, and a rich vocabulary system. But it also inherits the Semantic Web's adoption challenges. JSON-LD, in particular, has been a barrier to adoption for developers who don't want to understand contexts, blank nodes, and named graphs. The spec addresses this pragmatically — it mandates multiple serialization formats and recommends not depending on the external JSON-LD context — but the cognitive overhead of RDF is real.

**The versioning vocabulary is the spec's most underrated contribution.** LDES doesn't just say "you can publish versions." It defines explicit predicates (`ldes:versionOfPath`, `ldes:versionCreateObject`, `ldes:versionDeleteObject`, etc.) that let a consumer mechanically derive CRUD semantics from an append-only stream. This is event sourcing, formalized in RDF, with W3C Activity Streams terms as the recommended vocabulary. When a consumer sees `ldes:versionDeleteObject as:Delete`, it knows to remove the previously inserted or upserted triples. This is precise, composable, and maps directly to existing data integration patterns. It's the kind of vocabulary design that rewards careful reading.

**The transaction support is underdeveloped.** The spec defines `ldes:transactionPath`, `ldes:transactionFinalizedPath`, and `ldes:transactionFinalizedObject` for grouping members into atomic transactions, but the semantics are thin. A transaction's members can span multiple nodes, but the spec doesn't address partial visibility (what if the consumer sees some transaction members but not the finalization member?), rollback, or isolation. The finalization flag is a boolean — there's no concept of "this transaction was explicitly aborted." For a production data integration system, this is a gap.

**What LDES gets right that most API specs miss:** The specification treats the synchronization algorithm as a first-class design problem. It defines state management (what to remember across runs), immutability detection (multiple signals with explicit precedence), and back-off behavior (which status codes trigger retry with what strategy). Most data APIs leave these decisions to the client implementer, and every client reinvents them differently. LDES says: here is the catalog of problems, here is the set of acceptable solutions, pick from this menu. That's what a specification *should* do.

**The relationship to the broader Semantic Web:** LDES represents a practical, engineering-focused strand of Linked Data thinking — less "let's connect all the world's knowledge" and more "let's make it possible to efficiently sync RDF datasets over HTTP." This is the same pragmatism that produced TREE (the hypermedia spec LDES builds on), Solid (Tim Berners-Lee's personal data store protocol), and DCAT (data catalog vocabulary). It's the Semantic Web community learning from REST, event sourcing, and stream processing rather than insisting that SPARQL endpoints are the only answer.

---

## Relationship to Existing Wiki Themes

LDES connects to several patterns the wiki already tracks. The append-only immutable log is the same architectural move as [[The Log is the Agent]], but applied to data publishing rather than agent cognition. The version-of-path + create/update/delete vocabulary is event sourcing for RDF, directly parallel to the bi-temporal patterns in [[Event Sourcing — Set-and-Remove Bi-Temporal Events]] — LDES even has the same effective-time-vs-publication-time distinction with `ldes:timestampPath` vs `ldes:versionTimestampPath`. The polling-based synchronization model is the reconciliation backstop that [[Event-Driven vs Polling Architectures]] argues every production system needs, formalized as a specification rather than an ad-hoc script. And the RDF graph structure with typed edges, SHACL shapes, and named graphs is the non-AI version of what [[Context Graphs]] proposes for agent memory — structured knowledge representation with explicit relationships rather than embedding similarity.

---

*Sources: [[raw/index-html]], [[summary/index-html]]*
*Last updated: 2026-08-06*
