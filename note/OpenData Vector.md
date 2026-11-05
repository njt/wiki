# OpenData Vector

OpenData's MIT-licensed *Vector* is a stateless vector search engine built entirely on SlateDB (an LSM store over object storage): no leader election, no shard assignment, no node-to-node communication — every node can serve any data because all state, including metadata, lives in S3. The pitch is the gap between `pgvector` and a managed vendor, with a headline of 100M vectors for ~$350/month on a single reserved instance.

---

## Precis

The post's organising frame is a three-generation history of online systems on object storage: **tiered** (cold data on S3), **disaggregated** (durable state on S3, but nodes still hold stateful shard assignments — Chroma, Milvus), and **stateless** (object storage as "only truth"). Vector claims to be the only generally-available OSS third-generation online vector database, with Turbopuffer as the proprietary analogue.

Three decisions make statelessness work for vector search:

- **IVF/SPANN inverted indexing** (from SPFresh) over graph indexes like HNSW. Graph traversal is node-by-node sequential GETs against storage with up-to-100ms first-byte latency; IVF batch-loads whole posting lists per round trip, and incremental index maintenance is far cheaper.
- **LIRE compaction on an LSM tree.** Object storage forbids read-modify-write, so ingestion is append-only, with reconciliation deferred to compaction and a lazy SlateDB-merges adaptation of LIRE minimising write amplification.
- **Share-everything.** Nodes never communicate; readers operate on SlateDB snapshots and need no external consistent database.

Deployment topologies run from embedded-in-your-process to buffered ingest, with the argument that statelessness makes a single Kubernetes pod a legitimate production target. Stated tradeoffs: warm queries ~10ms vs sub-millisecond HNSW; write acknowledgement upward of a second (batched to amortise PUT costs); cold queries sub-second where a cold HNSW graph can take dozens of seconds. Benchmarks: low single-digit-ms warm latency at 90% recall, ~1K–12K writes/sec, S3 costs in the tens of dollars.

## Key Quotes

> There's growing consensus in the database community that object storage is a "good thing". The 99.999999999% durability SLAs, the cost efficiency (1/4 storage cost, $0 cross-AZ networking), and the strong consistency are all distributed system nightmares that are solved by object storage.

The strongest claim in the piece, and it reframes the whole architecture conversation: object storage doesn't just move the cost curve, it *deletes whole subproblems* (replication, durability, consistency) that second-generation systems still engineer around.

> It is challenging to adapt a second generation system to be truly *stateless*... As far as we are aware, OpenData *Vector* is the only OSS third generation, online vector database that is generally available.

A deliberately narrow "as far as we are aware" — but the point that statelessness must be designed in from the ground up, not retrofitted, is the load-bearing argument for why the field hasn't already converged.

> IVF, despite its relatively imprecise retrieval algorithm, can batch load data per round trip. This more than makes up for the increased compute needs on cache misses.

The cost-model swap in one sentence: trade compute you have for round trips you can't afford. It is the exact same move as Infino's, applied to ANN instead of BM25.

## Key Themes

#concept — generation taxonomy of object-storage architectures (tiered → disaggregated → stateless) #pattern — trading compute and recall for round trips #tool — Vector, SlateDB, SPFresh/LIRE #concept — statelessness as an operational simplicity argument, not just a cost argument

## Analysis

This is vendor content, and the benchmark section is candid enough to mostly earn its claims: the ~10ms warm latency, the ~1s write acknowledgement, and the no-read-your-writes-after-Buffer caveat are the kind of tradeoffs vendors usually bury. What's genuinely interesting is not the engine but the thesis it carries: the third-generation stateless architecture is a *simplification* argument, not just a cost argument. Coordination, rebalancing, leader election — the entire operational tax of distributed databases — is traded for a single node plus S3 that "never loses data." Whether that holds at real read QPS, not 32 queries/s, is the unanswered question, and the benchmark section's own framing (p90 cold latency dominated by the slowest posting-list fetch) hints that the tail is the story.

The other notable thread is the second time in this wiki we've seen the same insight from a different direction: on object storage, the scarcest resource is round trips, and the index must be shaped around batch loading. That this now appears independently in both full-text search (Infino) and vector search (Vector) suggests it is converging on the status of a settled design law for the storage layer of the agent era — which is a bigger deal than any single engine.

## Relate

This source sharpens [[Infino — Search Index on Object Storage]]'s round-trip-economics thesis by applying the same cost-model swap to ANN indexing, where the tradeoff lands differently: batch-loaded posting lists versus graph traversal, with cold-query latency as the surprising win. It complicates [[zvec]]'s "SQLite of vector databases" framing from the other end — zvec argues most vector search fits in one machine's memory; Vector agrees about one machine but makes that machine's durability live in object storage rather than a local WAL, pushing the single-node argument to cloud-native scale. And it gives [[RIP Vector Database]] a concrete datapoint: if a stateless engine can be DIY-operated at $350/month for 100M vectors, the "vector database as managed product" category the post positions against is under real pressure from below.

---
*Sources: [[raw/introducing-vector]], [[summary/introducing-vector]]*
*Last updated: 2026-11-06*
