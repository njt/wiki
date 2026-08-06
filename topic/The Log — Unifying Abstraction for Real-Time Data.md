# The Log — Unifying Abstraction for Real-Time Data

Jay Kreps' 2013 manifesto arguing that the append-only, totally-ordered log is the single most important and most underappreciated abstraction in software engineering — present at the heart of databases, replication, consensus, data integration, and stream processing, yet invisible to most practitioners. The article is both a technical survey and a design argument, grounded in Kreps' experience building Kafka at LinkedIn. It has become one of the most influential posts in distributed systems engineering.

---

## What Is a Log?

Kreps defines the log with almost absurd simplicity: "an append-only, totally-ordered sequence of records ordered by time." Each entry gets a unique sequential number that acts as a logical timestamp, decoupled from any physical clock. This decoupling is essential for distributed systems, where wall-clock time is unreliable.

> "A log is perhaps the simplest possible storage abstraction."

The log is distinct from application logging (syslog, log4j) — those are degenerative, human-readable forms. The data log is built for programmatic access and structured for machines.

### Logs in Databases

The log's original purpose was crash recovery: write intended changes to the log before applying them to data structures. The log is the authoritative record of what happened; tables and indexes are projections of that history. This is the architecture described in [[ARIES — Write-Ahead Logging Recovery]] — the WAL as the source of truth, everything else derived. Over time, databases discovered that the sequence of changes needed for recovery is exactly what's needed for replication: Oracle, MySQL, and PostgreSQL all ship logs to replicas. The log evolved from an ACID implementation detail to a data subscription mechanism — "almost by chance," Kreps notes.

### Logs in Distributed Systems

Kreps articulates the **State Machine Replication Principle**:

> "If two identical, deterministic processes begin in the same state and get the same inputs in the same order, they will produce the same output and end in the same state."

This is the conceptual core of the article. The distributed systems problem — making multiple machines do the same thing — reduces to implementing a consistent distributed log. The log squeezes nondeterminism out of the input stream. Kreps is refreshingly modest about this: "there is nothing complicated or deep about this principle: it more or less amounts to saying 'deterministic processing is deterministic.' Nonetheless, I think it is one of the more general tools for distributed systems design."

He connects this to the consensus literature: Paxos (via Multi-Paxos), ZAB, RAFT, and Viewstamped Replication all model the problem of maintaining a distributed, consistent log. His prediction that "the log will become something of a commoditized interface" has largely come true — [[Meerkat — QuePaxa Consensus at Cloudflare]] is a modern example of a consensus service built around a distributed log.

### Tables and Events Are Dual

> "If you have a log of changes, you can apply these changes in order to create the table capturing the current state."

A table is data at rest; a log is data in motion. The log is more fundamental: from the complete log of changes, you can recreate not just the current table but every previous state. This is the insight behind event sourcing ([[Event Sourcing — Set-and-Remove Bi-Temporal Events]]) and source control — `git` is essentially a log of patches. Kreps credits Datomic for productizing the log-centric database, but notes the idea had been in the literature for over a decade.

## Data Integration: The O(N²) Problem

Kreps frames data integration as a Maslow's hierarchy: reliable data flow is the base of the pyramid, yet most organizations have "huge holes" there while wanting to jump to advanced modeling. Two trends make this harder: the rise of event data (clicks, impressions, machine metrics — "several orders of magnitude larger than traditional database uses") and the explosion of specialized data systems (OLAP, search, graph, batch, key-value — each with its own data ingress).

The naive approach — custom pipelines between every pair of systems — yields O(N²) connections:

> "This clearly would take an army of people to build and would never be operable."

The log-based alternative: every data source publishes to its own log. Every subscriber reads independently, at its own pace. Adding a system means connecting it to one pipeline, not N consumers. This architecture:

- **Decouples producers from consumers.** The subscriber doesn't know or care whether data came from an RDBMS, a key-value store, or an application log.
- **Provides a universal clock.** Each subscriber's position in the log is a precise "point in time" — to avoid stale reads from a cache, ensure the cache has replicated past the write's log entry.
- **Buffers asynchronously.** Hadoop can consume hourly; a real-time query system can consume up-to-the-second. Neither affects the other.
- **Enables crash recovery.** A subscriber that goes down for maintenance catches up from where it left off.

> "I use the term 'log' here instead of 'messaging system' or 'pub sub' because it is a lot more specific about semantics and a much closer description of what you need in a practical implementation to support data replication."

### ETL Is Two Things Conflated

Kreps argues that ETL conflates extraction/cleanup (liberating data from source systems) with restructuring (fitting data to a warehouse schema). These should be separated: the clean, integrated data repository should be available for real-time and low-latency use, not just batch warehousing. This has an organizational benefit too: data producers become responsible for providing clean, well-structured feeds to the central log, rather than dumping the extraction burden on a central data warehouse team that can never scale to match the rest of the organization.

This architecture is now conventional wisdom. [[Streambed]] implements exactly this pattern — Postgres WAL → log-structured pipeline → Iceberg on S3 — as a single Go binary. [[Postgres CDC in ClickHouse, A Year in Review]] describes running this at hundreds-of-customers scale. [[Linked Data Event Streams (LDES)]] applies the same append-only log pattern to RDF data with formal synchronization semantics.

## Stream Processing Is Continuous Processing

Kreps reframes stream processing as something much broader than the "SQL engine for events" niche:

> "Stream processing is infrastructure for continuous data processing. The computational model can be as general as MapReduce or other distributed processing frameworks, but with the ability to produce low-latency results."

The historical argument: batch processing is a relic of manual data collection. The US Census is batch because it involved riding around on horseback. As data collection becomes continuous, processing naturally becomes continuous. "Production 'batch' processing jobs that run daily are often effectively mimicking a kind of continuous computation with a window size of one day."

The log enables this because it makes every dataset multi-subscriber and ordered. Stream processing jobs read from logs and write to logs, forming a graph of processing stages. Derived feeds are indistinguishable from primary feeds — consumers don't know or care whether data is raw or computed. This composability is the killer feature.

## Building Kafka at Scale

Kreps describes the LinkedIn origin story: after building a key-value store, he tried to get Hadoop working for recommendations. "Having little experience in this area, we naturally budgeted a few weeks for getting data in and out, and the rest of our time for implementing fancy prediction algorithms. So began a long slog." Data copying dominated development. Each new source required custom configuration. The pipeline was "the source of a huge number of errors and failures."

This led to Kafka, designed to combine messaging semantics with the database log concept. Three scaling techniques:

- **Partitioning**: each partition is a totally ordered log; no global ordering between partitions. Append throughput scales linearly with cluster size.
- **Batching**: from client-to-server through disk writes, replication, and consumer delivery — the log is optimized for linear read/write patterns.
- **Zero-copy**: a single binary format maintained from in-memory log through on-disk and network transfer.

As of 2013: 60 billion unique message writes per day. Kreps notes, with characteristic understatement, that "a sign you've created a good infrastructure abstraction is that AWS offers it as a service" — referring to Amazon Kinesis.

## Critical Analysis

**The article's influence is hard to overstate.** It shaped how a generation of engineers thinks about data infrastructure. The event-driven architecture pattern it describes — publish events to a log, have consumers subscribe independently — is now the default for any organization operating at scale. Kafka became a billion-dollar company (Confluent, which Kreps co-founded). The ideas are so embedded that it's easy to forget they needed arguing in 2013.

**What's aged well.** The prediction that logs would become commoditized infrastructure was prescient. The separation of data cleanup from warehouse restructuring anticipated the modern data mesh. The reframing of stream processing as generalized continuous processing (not a SQL-engine niche) was ahead of its time — tools like Flink and ksqlDB vindicated it, and Samza (which Kreps' team open-sourced) directly implemented these ideas.

**What's aged less well.** The article ends mid-sentence in this capture (the original continues into Part Four on system building). Kreps' framing is deeply batch-vs-stream, and the modern landscape has largely settled on a both-and approach — the Lambda architecture was an attempt at this, and while it's now unfashionable, the practical reality is that most organizations run both. The article also predates the rise of event sourcing as an application pattern (not just infrastructure), the serverless event-driven architectures of AWS Lambda + Kinesis, and the "database as log" designs of systems like [[celld]] and FoundationDB.

**The organizational argument is underrated.** Kreps' point about aligning incentives — make data producers responsible for clean feeds, don't dump it all on a central team — is as much about organizational design as technical architecture. It anticipated the data mesh's "data as a product" principle by half a decade. Most technical readers fixate on the log mechanics and miss this.

**The State Machine Replication Principle as the Rosetta Stone.** This is the article's deepest contribution: the insight that a whole class of distributed systems problems — replication, consensus, data integration, stream processing — are all the same problem viewed from different angles. Once you see the log as the unifying abstraction, everything else becomes a projection. This idea directly informs [[The Log is the Agent]], which applies the same pattern to AI agent architecture, treating the agent's event log as primary and all other state (graph, tools, outputs) as deterministic projections.

**What the capture misses.** The archived version is truncated — the original LinkedIn post continues with Part Four ("System Design") and additional sections. The missing material covers using logs for system design patterns and practical implementation details. For the complete argument, the original LinkedIn Engineering post should be consulted. That said, even this truncated version contains the essential ideas that made the article influential.

## Key Themes

#log #distributed-systems #data-integration #stream-processing #kafka #event-sourcing #database-internals #fundamentals #concept

---

*Sources: [[raw/oifml]], [[summary/oifml]]*
*Last updated: 2026-08-06*
