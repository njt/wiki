# Postgres to Snowflake Data Mirroring

Snowflake's engineering deep-dive into **data mirroring** — a public-preview feature that replicates Postgres into Snowflake by *pushing* change batches out of Postgres (via a `snowflake_cdc` extension) directly into Iceberg tables, then applying them transactionally and serverlessly. The core move is inverting CDC from pull to push, and from an external consumer that "knows nothing about the state of Postgres" to an extension that lives inside it.

---

## Key Quotes

> "transactional push into the data lake, transactional apply in Snowflake, no extra infrastructure"

The thesis compressed to a slogan. The "no extra infrastructure" is the part that lands hardest: no Debezium, no Kafka, no connectors that can fall behind. The object store becomes the decoupling layer — producer and consumer each read and write at their own pace, with S3 as the buffer.

> "The solution to this problem is quite simple: Push the changes from Postgres into a data lake."

Snowflake frames push-vs-pull as the *reason* CDC pipelines are fragile. A pull-based consumer reads WAL over the network but can't tell whether Postgres is down, whether a schema changed, or how a snapshot aligns with changes. An extension *inside* Postgres can coordinate all of that because it's party to the transaction machinery.

> "When we make replication a transactional process that's controlled from Postgres … we can make a stream of perfect deletions and insertions that are applied exactly once."

This is the sharpest technical claim in the piece, and it's the payoff of the push model. Because Postgres decides what goes in each batch and Snowflake applies whole batches in one transaction, the system never needs the upsert trick — which the article argues is expensive on columnar storage and produces inconsistent intermediate states. Inserts get appended, never matched against the target table.

> "With live views, it is no longer necessary to apply changes very often to have low lag."

The elegant cost-reduction move. Unapplied change-log rows are unioned with the target table at query time, with filters pushed down into both Parquet and base-table scans. You get sub-minute lag without paying to apply every batch immediately.

> "You set it up once. It runs forever."

The marketing echo of the article's real ambition: turn replication from a process with "many complex failure conditions" into "a Swiss clock."

## Key Themes

#concept #tool #pattern #comparison

- **#concept** — **Push-based CDC.** Inverting the conventional direction (external consumer pulls WAL) eliminates a whole class of infrastructure and, critically, gives the replicator *knowledge of Postgres state* — when schemas change, how snapshots align with changes, whether the DB is alive.
- **#concept** — **Transactions as a replication primitive.** The article echoes the [[Postgres Transactions Are a Distributed Systems Superpower]] thesis: transactions hide system/hardware failure, and ETL/CDC pain is "what happens when transactions go out the window." Mirroring restores transactions on *both* sides of replication.
- **#tool** — **Snowflake Postgres** (`snowflake_cdc` extension), **Apache Iceberg** + compressed Parquet, **pg_lake** (open-source "Postgres for your data lake"). The whole thing sits on top of Iceberg's open format rather than a proprietary wire protocol.
- **#pattern** — **Meta log as control plane.** Changes land as per-table change logs plus a "meta log" of instructions; the Snowflake apply process is a finite state machine executing that log. This is the [[The Log — Unifying Abstraction for Real-Time Data]] architecture, with the log living in Iceberg.
- **#comparison** — **Mirroring vs. Postgres-for-your-data-lake.** Snowflake now offers two unification paths: always-on automatic replication (mirroring) versus developer-controlled SQL movement into Iceberg. The former for high-frequency replication, the latter for flexible ETL.

## Critical Analysis

**The push inversion is genuinely novel — and genuinely Snowflake-locked.** The idea that a Postgres extension should push changes to object storage, decoupling producer and consumer, is a real architectural contribution. But note the asymmetry: the extension is Snowflake's, the apply process is Snowflake's, and the feature only targets Snowflake's own Postgres service. Compare [[Streambed]] (pull-based, Iceberg-agnostic, single Go binary) and [[Artie]] (managed CDC to *any* warehouse). Snowflake is making the same "we pre-solved the edge cases" bet as Artie, but only for customers who've already bought Snowflake Postgres.

**"It runs forever" is doing a lot of quiet work.** The article barely mentions failure: it notes WAL can be re-snapshotted after unexpected loss, and that failover slots make this "very rare in practice." That's the kind of sentence [[Postgres CDC in ClickHouse, A Year in Review]] spends a whole year contradicting — the long tail of schema-change gaps, slot backpressure, and long-running transactions is where CDC systems live or die. A push model *should* reduce that tail (the extension knows about schema changes as they happen), but the article asserts it rather than demonstrates it.

**Live views are the smartest idea here, and they're under-explained.** Decoupling "apply frequently" from "low lag" is the correct optimization: batch-apply cheaply, and pay a small read-time overhead only for the unapplied delta. But the article gives no numbers on that overhead, what happens to live-view performance as the change log grows between applies, or how filters interact with Iceberg metadata. The mechanism is sound; the magnitude is a hand-wave.

**The "perfect deletions and insertions" claim deserves scrutiny.** "Exactly once" across a Postgres transaction boundary and a Snowflake transaction is only as exact as the coordination between them. The article says the apply process moves tables forward "exactly to a Postgres transaction boundary" — but the mechanism for discovering *which* boundary, and what happens if Snowflake applies a batch whose Postgres transaction later aborts, is left unspecified. The push model helps; it doesn't make the two-phase problem disappear.

**The strategic read.** This is Snowflake collapsing the operational-vs-analytical gap it used to depend on third parties (Fivetran, [[Artie]], PeerDB) to bridge. "Postgres for your data lake" plus "data mirroring" is a land-grab on the Postgres-to-warehouse pipeline — and, notably, on the Iceberg ecosystem, since the destination is open-format tables rather than Snowflake's proprietary storage. The open format is the wedge; the extension is the moat.

---

## Related Pages

- [[Streambed]] — Postgres-to-Iceberg CDC in a single Go binary. The pull-based, infrastructure-free counterpoint: same destination (Iceberg on S3), opposite direction of control.
- [[Artie]] — Managed Postgres→Snowflake CDC. The third-party product Snowflake's own mirroring now competes with, and the "we pre-solved the edge cases" pitch Snowflake borrows.
- [[Postgres Transactions Are a Distributed Systems Superpower]] — the transactions-as-superpower thesis that mirroring extends from co-located workflow state to cross-system replication.
- [[The Log — Unifying Abstraction for Real-Time Data]] — the log-centric architecture; mirroring's change logs + meta log in Iceberg is Kreps' blueprint with the log relocated into object storage.
- [[Postgres CDC in ClickHouse, A Year in Review]] — the field report on why CDC is "hundreds of edge cases"; the counterweight to mirroring's "it just runs forever."

---

*Sources: [[raw/postgres-to-snowflake-replication-mirroring]], [[summary/postgres-to-snowflake-replication-mirroring]]*
*Last updated: 2026-08-14*
