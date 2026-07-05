# Lakebase and LTAP

Databricks co-founder Reynold Xin lays out a two-act database architecture: Lakebase makes Postgres compute stateless by externalizing the WAL and data files into distributed cloud services, and LTAP (Lake Transactional/Analytical Processing) eliminates CDC pipelines entirely by storing operational data once in open columnar formats that both Postgres and Lakehouse engines read directly. The storage layer becomes the unification point, not the engine — a deliberate inversion of HTAP.

---

## The Monolith Diagnosis

Xin opens with a crisp diagnosis of why traditional databases are fragile: "the WAL exists to make *writes* fast (and safe), and the data files exist to make *reads* fast." The problem is both live on the same machine. Disk dies, data dies. Read replicas are physical clones that take forever to provision. HA needs synchronous replication. Analytics queries contend with OLTP on the same hardware.

> "OLTP databases were far from a solved problem: they were clunky, difficult to scale, and incredibly fragile."

Commentary: This is the kind of thing you can only say after shipping a competing product. It's also correct. The monolith database has been the default for so long that its fragility is invisible — until you try to run a SaaS business on one.

## Lakebase: Stateless Postgres

The fix is conceptually clean: externalize the two on-disk components into independently scalable services.

**SafeKeeper** replaces the WAL with Paxos-based quorum replication across distributed nodes. A commit is durable when replicated across the quorum, not when a single local disk claims to have flushed. This eliminates both the misconfiguration risk ("the operating system might even decide to lie to you about flushing") and the node-loss risk.

**PageServer** replaces data files. It streams the WAL from SafeKeeper, asynchronously applies changes, and materializes pages into cheap cloud object storage. Postgres's buffer pool and a local disk cache sit in front, so "for the vast majority of operations, read latency is indistinguishable from a monolith."

> "A commit is durable once it is replicated across SafeKeeper nodes via Paxos, not when a single local disk claims to have flushed it."

The upshot: unlimited storage, elastic compute (scale to zero), simpler HA, and instant branching — "a branch or a clone is a metadata operation rather than a physical copy."

Commentary: This is the part that builds on Neon's architecture. The innovation isn't separating compute from storage — that's been done. It's doing it for Postgres in a way that preserves wire compatibility, extensions, and the full SQL surface. The "still Postgres" constraint is the hard part.

## LTAP: One Copy, No Pipelines

The real contribution is LTAP. Once data lives in externalized storage, why keep two copies — one for transactions (row format) and one for analytics (columnar format) — connected by a brittle CDC pipeline?

> "In LTAP, there is nothing to opt into."

PageServer transcodes Postgres data from row format into Parquet columnar layout as it materializes pages. Exact Postgres value representation is preserved "down to the bits" — exotic types like NaN, ±Infinity, and extended-precision NUMERICs go into a structured overflow field that's "both directly queryable by any engine and sufficient to reconstruct the original Postgres bytes exactly on the way back."

MVCC intermediate row versions are preserved for PITR but hidden from analytical readers and eventually garbage-collected.

**Freshness:** When an analytical query starts, it asks Postgres for the current LSN — "a cheap metadata lookup." The analytical engine reads the bulk of data from object storage, then merges in the small set of very recent changes from PageServer. "Postgres itself serves none of the analytical read traffic other than returning a single number (LSN)."

> "There is no list of replicated or mirrored tables, because there is no replication."

Commentary: This is a clean kill shot at the entire CDC industry. [[Streambed]], [[Artie]], and every other pipeline tool exist because databases don't natively expose their data in analytical formats. If LTAP works as described, those tools become unnecessary for Lakebase users — the data is already where analytics needs it, in the format it needs.

## The HTAP Rebuttal

Xin takes explicit aim at HTAP, arguing no widely adopted HTAP system exists for three reasons: incomplete feature set (building one engine that's good at everything takes decades), no ecosystem (Postgres and Spark each have vast ones), and no performance isolation (same hardware, same contention problem).

> "All three problems above trace back to the same root cause: unifying the two workloads into one engine."

LTAP sidesteps all three by unifying at the storage layer instead. Postgres owns transactions; Lakehouse engines own analytics. The data underneath is one governed copy.

## Critical Analysis

**This is a Databricks product announcement wearing a technical architecture essay.** That doesn't make it wrong, but it does mean the sharp edges are sanded off. Several questions go unasked:

**What's the latency from commit to columnar materialization?** Xin says analytical queries read "fresh" data by merging recent changes from PageServer, but doesn't quote a materialization SLA. If the merge path is fast enough, this doesn't matter. If it isn't, you've just replaced a CDC pipeline with a different synchronization mechanism that has its own latency characteristics.

**The type system overflow field is doing a lot of work.** "Directly queryable by any engine" is a strong claim — the exotic values that need this field are exactly the ones that break cross-engine compatibility. How does a Spark SQL query handle a Postgres `NUMERIC(38,10)` that overflowed to the canonical text field? The answer likely involves Databricks-specific integration, not pure open-format magic.

**The dual-write verification period is telling.** During rollout, both row and columnar formats are written "for data verification." This is a confession that the transcoding path isn't trusted yet, which is the right engineering posture. But it also means the columnar path hasn't been battle-tested in production at scale.

**"Every table, automatically" sounds great until you think about cost.** Columnar compression saves money, but transcoding every table into Parquet is compute you're paying for. For append-only event tables, this is a clear win. For tables with heavy update/delete patterns, the MVCC version churn means you're materializing and then garbage-collecting many intermediate versions.

**The architecture is genuinely elegant.** Externalizing the WAL and data files into independent services, then using the materialization path to transcode into analytical formats, is a clean design. The benefits Xin lists — unlimited storage, elastic compute, instant branching — do follow "almost mechanically" from the architecture. That's the mark of a good design: the important properties emerge from the structure rather than being bolted on.

**The CDC-killer framing is also a cloud lock-in play.** "No replication" means your operational data lives in Databricks' object storage in Databricks' format. The open formats (Delta/Iceberg, Parquet) are technically portable, but the SafeKeeper→PageServer→Lakehouse integration is a Databricks stack. That's fine — it's their product — but "no more pipelines" means "you're on our platform now."

## The Storage-Layer Unification Pattern

LTAP is an instance of a broader pattern worth watching: **unification at the storage layer rather than the engine layer.** This is the same move [[The Limits of Generalized Sync]] identifies in the sync engine space — read paths generalize; write paths resist. LTAP solves this by making the storage layer the generalization point and keeping write-path engines specialized.

It also connects to [[Databases and Data]]'s observation about convergent architectures: every database wants to be the one database you need. LTAP inverts that — instead of one database doing everything, it's one storage format serving multiple specialized databases.

The [[Smart Models Dumb Pipes]] resonance is accidental but real: Postgres and Spark are the "smart" engines; the storage layer is the "dumb pipe" that connects them. The pipe is dumb in the best sense — open formats, no transformation logic — and the engines are smart where they need to be.

---
*Sources: [[summary/lakebase-ltap-rethinking-database-storage.md]]*
*Last updated: 2026-07-05*
