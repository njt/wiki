# DuckLake Performance Tuning (MotherDuck Webinar)

A MotherDuck webinar (Alex Monahan and Matt Martin, co-authoring a DuckLake book due early 2027) on making the DuckLake lakehouse both faster and cheaper. Its central thesis is architectural, not tactical: DuckLake's SQL-database catalog is what makes interactive low-latency lakehouse queries possible at all, because Iceberg and Delta "put their metadata in the wrong spot." The tuning advice then falls out of the three-bucket architecture (catalog, storage, compute), with a recurring method — log everything, then hand the JSON to an AI agent.

---

## The architectural claim

The headline argument is latency arithmetic rather than a benchmark: reading from Iceberg requires four serial steps (catalog → manifest list → manifests → parquet), each potentially ~100ms on object storage, which is built for throughput, not latency. DuckLake makes one ~5ms Postgres round trip, then reads parquet.

> "Iceberg and Delta, in my opinion, and I think the data is with me. They put their metadata in the wrong spot, put it on object storage, they should have put it the same place they put the catalog, which is a database."

Matt adds the four-decades argument: relational databases are a mature, well-understood, indexed, heavily optimized substrate; object stores are young and blunt. Four decades of tuning knowledge applies to the metadata layer only if the metadata lives in a database.

> "Most lakehouses feel like an archive. It feels like you're going in the library, you're having to relearn the Dewey Decimal system, you're walking down the dusty books, you're looking for it, finding it and grabbing it out."

The honest caveat, which the gist's own digest flags: this is assumed request counts, not a measured head-to-head. The rhetoric is stronger than the evidence.

## The three buckets as a tuning framework

Every optimization maps to catalog, storage, or compute. Rules of thumb by workload shape:

- **Big reads** → file layout: partitioning (skip whole files) and sorting (row-group pruning via parquet statistics)
- **Small reads** → catalog (it's hit proportionally more often)
- **Big writes** → compute (compression cost)
- **Many small writes** → inlining

Measure first, always: Postgres slow-query logs with explain plans, DuckDB profiling (HTTP times, scan/join times as JSON). And the sanity check: "is the juice worth the squeeze? Are you going to spend another six hours shaving off another two seconds on a batch job that runs once a day?"

## File layout: partitioning, sorting, space-filling curves

DuckDB 2.0 adds `PARTITION BY` and `SORT BY` in `CREATE TABLE`/CTAS. Partitioning works on a fixed set of transforms (year/month/day) precisely so the optimizer can reason about them — "hidden partitioning" means the analyst filters on the original column and the optimizer maps it onto the partition transform. Sort-by-expression (any scalar function) is claimed to be unique to DuckLake.

Alex's favorite optimization is Z-ordering/Hilbert curves: sequential sorts scramble the second column, but space-filling curves keep multi-dimensional locality, so "restaurants near me" style multi-filter workloads read one contiguous chunk. Matt contributes the operational gotcha: redefining partition/sort keys applies only to *new* files — retrofitting requires a CTAS rebuild (`CREATE OR REPLACE TABLE ... PARTITION BY ... SORT BY AS SELECT *`), which is cheap in lines of code but not in hours.

## Inlining: the killer feature

Inserts ≤10 rows (default) go into the SQL catalog, not parquet: lower latency, no small-file problem, fully transactional and instantly readable — unlike a Kafka buffer. Flush at roughly your target parquet file size (~1M+ rows); the flush respects partitioning and sorting. It also eliminates most compaction work. This only exists *because* the catalog is a real database — the architecture pays for itself here, not just in read latency.

## Parquet settings and agent-driven tuning

Three settings: parquet v2 (a decade old, just use it), zstandard over snappy (more Pareto-optimal), and row-group size 2–4x, targeting ~10 row groups per file — the file-size-to-row-group ratio matters because the settings are coupled. Alex's prescribed method: have an agent run a **design of experiments**, varying settings jointly.

> "If the agent tries to tune these sequentially, you will end up in a suboptimal local optima, not a global optima... Now we can assume things are more interrelated rather than assume independence... we can brute force the heck of it in the modern times."

Result: ~40% improvement, "not a 10x" — and Matt's old-school one-at-a-time instinct gets lovingly mocked: "this isn't 2009 where you were just writing VBA code with SQL servers in Excel." Wide tables (sensor data) may flip the compression advice to uncompressed.

## Catalog tuning and maintenance

Index the column-statistics table (the biggest catalog table); size the DB so the whole catalog fits in RAM — a petabyte-scale lakehouse catalog is only a few gigs, so skip the "$1 postgres." Expire snapshots with a progressive stair-step (daily → weekly → monthly), two SQL commands. Maintenance is mandatory, unlike a warehouse: compact deletions, compact files to ~1GB, and keep a 10-day deletion window so a vacation doesn't force cold retrieval of logically-deleted-but-still-billed files. DuckDB 2.0's asyncio (~4 background threads per core for object-storage reads) gives ~3x faster bulk reads, corroborated only by an anonymous chat comment from a production workload.

## Themes

#concept — metadata placement as the architectural fork #tool — DuckLake, DuckDB 2.0, Postgres #pattern — design-of-experiments tuning with agents as the modern replacement for one-knob-at-a-time craft

## Opinion

This is vendor content — a MotherDuck webinar co-hosted by the DuckLake book's authors — and it shows: the Iceberg comparison is arithmetic, not measurement; the 40% figure is the speaker's own six-month-old testing; concurrency, governance, and catalog high-availability are never addressed, and the Hive Metastore question was deferred and never answered. But the core argument survives the bias: putting lakehouse metadata in a relational database is a genuinely sound architectural bet, and inlining is the feature that could only exist under that bet. The most interesting thing in the transcript is incidental: the tuning methodology has fully absorbed agents. "Point your agent at the JSON logs" and "have the agent run a design of experiments" are stated as the *normal* way to tune a data system in 2026 — statistical thinking (interdependence, global optima) returned to practice because brute-force search got cheap. That is a bigger claim about how engineering works now than anything about parquet row groups.

## Related pages

This strengthens [[DuckLake — SQL as a Lakehouse Format]]'s architectural pitch with concrete tuning practice and the quantified Iceberg-latency argument, and extends it from "what DuckLake is" to "how you make it fast." It nuances [[Iceberg Table Cleanup]] — that page's snapshot-expilation and orphaned-file concerns reappear here as catalog maintenance and write-contention cleanup, now with stair-step expiration schedules. It connects to [[What Is Columnar Storage (MotherDuck)]]: row-group pruning, parquet v2, and zstandard are exactly that page's columnar mechanics applied at lakehouse scale. And it is a clean worked example of the agent-as-analyst loop in [[Agent Coding Workflow]] — measurement first, agents shredding JSON logs and running experiments, humans keeping the "is the juice worth the squeeze" judgment.

---
*Sources: [[raw/c6ca4aa29f37bb3f1e818f5362b1915f]], [[summary/c6ca4aa29f37bb3f1e818f5362b1915f]]*
*Last updated: 2026-11-03*
