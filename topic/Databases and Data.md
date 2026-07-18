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

[[Replace Athena with DuckDB (Lambda)]] demonstrates DuckDB-on-Lambda as a 75-86% cheaper alternative to Athena for S3 Parquet analytics. The cost-model inversion: Lambda charges per GB-second of compute, Athena charges $5/TB scanned. DuckDB's httpfs extension reads only relevant Parquet columns via HTTP range requests, so Lambda execution time grows sub-linearly with data volume while Athena's scan cost grows linearly.

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

**Database migration patterns.** If you start with SQLite and outgrow it, what's the migration path to Dolt or AliSQL? The interop story between these databases is mostly undocumented. [[SQLite Is All You Need]] names the breakpoint (many writers contending on the same rows, need for read replicas, real analytics over hundreds of millions of rows) but, like [[SQLite is All You Need for Durable Workflows]], doesn't describe the migration experience itself.

**Database reliability and failure modes.** [[How to Corrupt an SQLite Database]] is the only page tackling the question of *when* database guarantees fail — the boundary between the library's promises and the environment's betrayals. A gap worth filling for other databases.

## Key Themes

#databases #convergence #data-quality #version-control #vector-search #agent-data

## Pages

- [[AliSQL]] — Alibaba's MySQL fork: DuckDB columnar OLAP + native vector search. 200x speedup
- [[Radicle]] — P2P sovereign code forge built on Git. Cryptographic identity, gossip protocol, no central server. The most serious decentralized GitHub alternative
- [[Dolt]] — SQL database you can fork, clone, branch, merge. Git + MySQL
- [[bucketvcs]] — Git server backed directly by cloud object storage: single Go binary, the bucket IS the repository, no database holds Git objects
- [[Graft]] — SQLite replicated to the edge via object storage
- [[SQLite is All You Need for Durable Workflows]] — SQLite + Litestream is the right default for agent workflow state; Postgres is the upgrade path, not the starting line
- [[SQLite Is All You Need]] — DB Pro's benchmarked case for SQLite as a production web backend: 3,654 req/s on one file, WAL mode quantified, and the argument that user acquisition is the bottleneck, not database throughput
- [[How to Corrupt an SQLite Database]] — The SQLite team's exhaustive catalog of corruption failure modes: the trust boundary between library guarantees and environmental failures
- [[Write Snapshot Isolation]] — SI checks stale writes; WSI checks stale reads. Serializability in one fix
- [[Dapper Performance Trap]] — NVARCHAR vs VARCHAR implicit conversion defeats indexes. Quiet perf killer
- [[zvec]] — Alibaba's in-process vector DB. Billions of vectors, milliseconds, pip install
- [[Shaper]] — SQL-driven dashboards powered by DuckDB. Chart types via casting syntax
- [[sql-crack]] — VS Code extension: SQL queries as interactive execution flow diagrams
- [[SiftRank]] — LLM-based document ranking with pairwise comparisons and inflection detection
- [[Materialized Views Are Obviously Useful]] — Sophie Alpert: incremental view maintenance is obviously useful; databases should handle derived data, not application code
- [[Long Live Systems of Record]] — Jamin Ball: agents don't kill systems of record, they raise the bar. "Where does the truth live" is the only question that matters
- [[PgDog]] — PostgreSQL proxy combining connection pooling, load balancing, and sharding in one binary with zero application code changes
- [[Postgres CDC in ClickHouse, A Year in Review]] — Field report on PeerDB's first year inside ClickHouse: 400+ customers, 200 TB/month, and the surprising complexity of making CDC feel boring
- [[Metrics SQL]] — Rill Data's SQL dialect for querying a YAML-defined metrics layer. Transpiles to engine-native SQL with inferred GROUP BY, parameterized literals, and MCP server for AI agents
- [[We Replaced Redis with MySQL for Inventory Reservations]] — Shopify's move from Redis to MySQL for inventory reservations: one-row-per-unit, SKIP LOCKED, and the case that connection pool pressure is the real bottleneck
- [[DocDB — Stripe's Zero-Downtime Database]] — Stripe's internal MongoDB-based DBaaS: 2,000+ shards at 5M QPS. Zero-downtime data movement as platform primitive — resharding, version upgrades, and tenancy migrations are all the same operation
- [[SQL Fraud Patterns (Fixel Smith)]] — Six composable SQL patterns for transaction fraud detection: velocity, impossible travel, amount anomalies, suspicious merchants, off-hours, and window-function primitives. Fraud rules as WHERE clauses, not ML models
- [[BEAVER]] — First enterprise text-to-SQL benchmark from real private data warehouses. GPT-5.2 gets 10.8%; with all oracle hints, 30.1%. The gap between BIRD (82%) and enterprise reality is a chasm
- [[KTX Context Layer for Data Agents]] — Open-source context layer for data agents: git-versioned wiki + executable semantic layer, ingested from dbt/Looker/Metabase, served via 11 MCP tools. Pre-merge validation gates on all agent writes
