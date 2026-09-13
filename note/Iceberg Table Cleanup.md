# Iceberg Table Cleanup

A production-depth guide to Apache Iceberg table maintenance from LakeOps (a vendor building an autonomous Iceberg control plane). Its argument: Iceberg's append-only architecture makes cleanup mandatory, the four maintenance procedures exist and work, but the operational intelligence around them — sequencing, parameters, per-workload retention — is what teams actually lack, and its absence is how tables get corrupted and cloud bills double.

---

## Key Quotes

> "Iceberg uses an append-only architecture. Every write creates new files and a new snapshot. Nothing is overwritten, nothing is garbage-collected automatically. This immutability is what makes ACID transactions, time travel, and concurrent access possible — and it is what makes cleanup mandatory."

The whole article in one move. Immutability is simultaneously Iceberg's core feature and its recurring invoice: every property the format sells you (ACID, time travel, concurrent writers) is paid for later in garbage collection. The same trade exists in every log-structured system — the write path is cheap because the cleanup problem is deferred to the operator.

> "What Iceberg does not ship is the intelligence around them: when to run each one, in what order, with what parameters, on which tables, and how to avoid the half-dozen ways you can corrupt a table or silently delete live data by running them wrong."

The honest framing of the gap — and, transparently, the exact gap the vendor sells against. Everything before this sentence is generic operational truth; everything after leans toward "and LakeOps has it."

> "One mismatched scheme can delete your entire table's data in a single run."

The URI scheme failure (`s3://` in metadata vs `s3a://` from a storage listing) is the article's best horror story, because the orphan-cleanup procedure does a *string comparison* to decide what's unreferenced. Every file looks orphaned; the deletion is indiscriminate. It's a reminder that "dumb tools with safe defaults" beats "smart tools with footguns" — this is a procedure that could refuse to run without a dry run, and doesn't.

> "OCC conflicts on streaming tables are not errors — they are expected."

A useful reframe buried in the streaming section. Compaction on a table taking three commits a minute *will* conflict; the design question is whether your retry budget, isolation level, and partition scoping absorb that as routine. The checklist that follows (retries 10–20, snapshot isolation, exclude hot partitions, partial progress) is the most practically valuable part of the piece.

> "Cleanup is not a scheduling problem — it is a continuous optimization problem."

The intellectual pivot and the product pitch in one sentence. Half true: parameters genuinely do drift as write velocity and query patterns change. But the catastrophes the article catalogs (URI mismatch, 1-hour orphan retention, `retain_last = 1`) are *static* misconfigurations that safe defaults would prevent — you don't need continuous optimization to refuse a destructive run.

## Key Themes

**#concept** — The snapshot chain as an unmanaged log: snapshots, orphans, manifests, and delete files all grow without bound because nothing is GC'd automatically.

**#concept** — Merge-on-read delete files: position deletes, equality deletes, and V3 deletion vectors (Roaring Bitmaps in Puffin files) as a compounding read-time tax that only compaction resolves — and stale min/max statistics that defeat data skipping even in V3.

**#pattern** — Expire → orphans → compact → manifests, with each dependency argued: expire first or orphans and compaction operate on the wrong file set; manifests last or compaction instantly re-fragments them.

**#pattern** — Detection → evaluation → execution: most teams jump straight to a blind nightly cron, which is how "the same parameters applied uniformly to tables with fundamentally different write patterns" corrupts tables.

**#tool** — Apache Iceberg, the four Spark SQL procedures, Flink checkpointing, Airflow DAGs, and LakeOps itself — running its pipeline on a purpose-built Rust/DataFusion engine instead of Spark.

## Opinionated Take

The operational content is genuinely good and mostly vendor-neutral: the sequencing arguments are correct and clearly reasoned, the streaming section (OCC, hot-partition exclusion, checkpoint alignment, partial progress) covers exactly what most guides skip, and the GDPR section is the rarest thing in lakehouse writing — it takes time travel seriously as a *compliance liability*, not just a feature. "Erasure latency = retention + compaction cadence, and you must document it" is advice that would surprise a lot of data protection officers.

Read the numbers as parables, though. "One team reported 38%," "approximately 200 TB across 324 tables," "95% faster at 90% lower cost" — all unattributed, all suspiciously precise, and the benchmark has no methodology. The Airflow strawman ("at 100+ tables it breaks") assumes a DAG per table, which is how nobody writes them; loops over a config table with per-workload parameter classes get you a long way. The structural truth the pitch rests on is real — maintenance is a detection problem before it's an execution problem, and static configs drift — but the distance between "this is hard" and "therefore buy the control plane" is covered by anecdote, not evidence. Take the checklist; audit the pitch.

## Related Pages

This source **strengthens** [[Postgres to Snowflake Data Mirroring]]: Snowflake's push-based CDC writes straight into Iceberg tables with "no extra infrastructure," and this is the invoice that arrives later — every mirrored `UPDATE` writes delete files, every failed batch risks orphans, every batch creates snapshots. Continuous CDC into Iceberg makes the cleanup pipeline part of the mirroring feature's real cost.

It **complicates** [[Lakebase and LTAP]]: LTAP's thesis is that open table formats become the operational system of record, read in place by many engines. This article is the counterweight — making open formats operational means owning garbage collection, compaction, and manifest hygiene that a closed database previously did for you. The lakehouse's storage-compute separation is not free; it's a maintenance subscription.

It **strengthens** [[Apache DataFusion]]: LakeOps is a shipping example of the "be the component, not the product" thesis — a purpose-built Rust/DataFusion engine doing compaction instead of Spark, with no JVM and no cluster to provision. DataFusion as the engine you build a maintenance plane on, not just query engines.

It **nuances** [[The Log — Unifying Abstraction for Real-Time Data]]: Kreps argued the log is the unifying abstraction, and Kafka shipped log compaction *with* the log. Iceberg's snapshot chain is a log too, but one where retention and compaction are left as an exercise for the operator — the "intelligence around the log" that databases normally internalize must be rebuilt outside the format, which is this article's entire subject matter.

---
*Sources: [[raw/iceberg-table-cleanup]], [[summary/iceberg-table-cleanup]]*
*Last updated: 2026-09-13*
