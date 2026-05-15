# Databases and Data

Storage is a design problem, not a commodity service. The pages in this wiki reveal a landscape where the boundaries between database categories are dissolving: MySQL gets columnar OLAP and vector search ([[AliSQL]]). Graph databases get vector search and temporal decay ([[NornicDB]]). SQLite gets edge replication ([[Graft]]) and version control ([[Dolt]]). The trend is convergence: every database wants to be the one database you need. Meanwhile, the foundational problems -- correctness, data quality, consistency -- remain as hard as ever and are getting harder as agents generate data at machine speed without human review.

---

## The Landscape

### Version-Controlled Data

[[Dolt]] is the clearest expression of "Git for databases": fork, clone, branch, merge at the data level. MySQL-compatible, with cell-level lineage tracking. Three deployment modes: standalone database, Git-like CLI, or MySQL replica that adds versioning to existing infrastructure. Being used for AI agent memory systems where agents need to branch, experiment with, and merge shared state.

[[Graft]] takes a different approach: SQLite replicated to the edge via object storage. Stateless, no cluster required, partial replication (fetch pages lazily). The comparison page maps the entire replicated SQLite landscape: mvSQLite (needs FoundationDB), Litestream (backup only), cr-sqlite (CRDTs for automatic conflict resolution), Turso/libSQL, rqlite (Raft consensus). Graft's "stateless on object storage" architecture is operationally simpler than anything requiring consensus nodes.

### Convergent Databases

[[AliSQL]] grafts DuckDB's columnar OLAP and native HNSW vector search onto MySQL. Zero learning curve -- use your existing MySQL tools. 200x analytical speedup on columnar queries. Battle-tested across millions of databases at Alibaba. The strategy is pragmatic: don't make people switch databases, just add the capabilities they need to the one they already run.

[[NornicDB]] unifies graph traversal, vector search, and temporal queries in one engine. Neo4j-compatible (Bolt/Cypher). The distinctive feature is Ebbinghaus-based memory decay: knowledge fades over time unless reinforced. This matters for AI agent memory, where an ever-growing store without decay overwhelms retrieval. 12-52x faster than Neo4j on benchmarks (self-reported).

[[zvec]] is Alibaba's "SQLite of vector databases" -- an embeddable library, not a service. Billions of vectors, milliseconds, `pip install`. Part of Alibaba's stack alongside [[AliSQL]]. The right choice when your vectors fit on one machine (and with modern hardware, that's billions).

### Data Quality

[[Correct by Construction]] treats clean data as a whitelist problem: decompose into anchors (IDs), attributes (values), and links (relationships), then enforce completeness, uniqueness, and no-NULLs at each level. Deliberately radical -- banning NULLs and JSON containers outright. The insight that sentinel values ("UNKNOWN", empty strings) are just NULLs in disguise is sharp.

[[Write Snapshot Isolation]] fixes a fundamental flaw in standard snapshot isolation: SI checks for stale writes when it should check for stale reads. WSI changes one line of code and achieves serializability. "Correctness should be structural, not bolted on" -- the same philosophy as Correct by Construction.

[[Anomaly Detection]] provides the monitoring layer: Welford's algorithm for running mean/variance in constant memory, hourly bucketing, 2-sigma threshold. No ML, no config. The right level of sophistication for catching data anomalies.

### Data Tooling

[[Shaper]] powers SQL-driven dashboards with DuckDB. Write SQL, get charts. The casting syntax for chart types (`::BARCHART_STACKED`) is clever but mixes presentation with query logic.

[[sql-crack]] visualizes SQL queries as interactive execution flow diagrams in VS Code. Column lineage, CTE expansion, performance scoring, workspace-wide dependency analysis. 100% local, no telemetry. Supports 13 SQL dialects.

[[SiftRank]] ranks any dataset by relevance to a natural-language prompt using LLM pairwise comparisons. Deterministic rankings from nondeterministic oracle in seconds for pennies. Useful for prioritizing sources in the [[LLM Wiki]] pattern.

[[DAB]] (Microsoft) auto-generates REST, GraphQL, and MCP endpoints over any database. The MCP server lets AI agents query databases through the Model Context Protocol with custom tool configuration. The "give agents database access safely" problem solved as middleware.

[[Dapper Performance Trap]] is a cautionary tale: NVARCHAR vs VARCHAR implicit conversion defeats indexes silently -- 176x slower on a million-row table. AI-generated Dapper code will produce this default every time because that's what the docs show.

### The LLM Data Pipeline

[[Data Engineering for Large Models]] is a 28-chapter open-source textbook covering the complete pipeline: pre-training data cleaning, multimodal alignment, synthetic data generation, RAG architecture, and enterprise DataOps. The "data-centric AI" framing: data quality improvements consistently outperform model architecture improvements at the margin.

[[GraphRAG]] (Microsoft) builds knowledge graphs from documents for structured retrieval. Four query modes: Global (corpus-wide), Local (entity-specific), DRIFT (hybrid), Basic (vector fallback). Expensive to build but significantly better than naive vector similarity for cross-document synthesis and holistic summarization.

## Key Tensions

**Convergence vs. composability.** [[AliSQL]] adds OLAP and vectors to MySQL. [[NornicDB]] adds vectors and temporal to graph. The appeal is obvious: one database for everything. The risk: jack-of-all-trades, master of none. Specialized databases will outperform converged ones on their specialty workloads. The question is whether the operational simplicity of one database compensates for the performance gap.

**Correctness vs. pragmatism.** [[Correct by Construction]] bans NULLs and JSON. [[Write Snapshot Isolation]] provides serializability at the cost of more aborted transactions. Both are correct in theory and friction in practice. Most teams will choose the pragmatic path (allow NULLs, accept SI's flaws) until they get burned. The frameworks exist for when they do.

**Version control for data vs. complexity.** [[Dolt]] proves you can branch and merge data. But version-controlled data adds significant cognitive and operational overhead. Most teams don't need it. For teams that do (ML experiments, regulatory compliance, agent memory systems), it's transformative.

**Local vs. distributed.** [[zvec]] and [[Graft]] both bet on "good enough on one machine." [[NornicDB]] and [[AliSQL]] scale to distributed deployments. The answer depends on data volume, but the trend toward local-first ([[Graft]]'s edge replication, [[zvec]]'s embedded library) suggests that many workloads are better served by simpler, local solutions.

**The Alibaba stack.** Three pages ([[AliSQL]], [[zvec]], [[OpenSandbox]]) come from Alibaba. They're building a coherent open-source data and AI infrastructure stack that rivals the Western equivalents. Worth watching as a portfolio, not just individual tools.

## What's Missing

**Agent-aware database patterns.** Agents generate queries differently from humans -- more repetitive, less optimized, prone to the [[Dapper Performance Trap]] class of errors. Database patterns specifically for agent-generated workloads don't exist yet.

**Data quality for agent-generated data.** [[Correct by Construction]] addresses human data pipelines. When agents are generating and storing data at machine speed, the quality problem changes character. Nobody has written the "Correct by Construction for agent-generated data."

**Streaming and real-time.** The wiki has batch-oriented databases (Dolt, AliSQL, NornicDB) and edge-oriented databases (Graft, zvec) but nothing on streaming data -- Kafka, Flink, real-time event processing. This is a gap worth filling, especially as agents generate event streams from tool use.

**Database migration patterns.** If you start with SQLite and outgrow it, what's the migration path to Dolt or AliSQL? The interop story between these databases is mostly undocumented.

## Key Themes

#databases #convergence #data-quality #version-control #vector-search #agent-data

---
*Synthesis of: [[AliSQL]], [[Dolt]], [[Graft]], [[Write Snapshot Isolation]], [[zvec]], [[Shaper]], [[sql-crack]], [[SiftRank]], [[Dapper Performance Trap]], [[Correct by Construction]], [[Anomaly Detection]], [[GraphRAG]], [[NornicDB]], [[Data Engineering for Large Models]], [[DAB]]*
*Last updated: 2026-05-14*
