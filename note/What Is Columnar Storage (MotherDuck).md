# What Is Columnar Storage (MotherDuck)

MotherDuck's measured guide to columnar versus row storage: the layout, the five stacked mechanisms that make analytical queries fast, the file-format landscape (Parquet, ORC, Arrow, table formats), and — just as importantly — the workloads where columnar is the wrong answer.

---

The pitch is one sentence: columnar storage packs every value of one column together instead of packing each row's fields together, so a query touching 3 of 50 columns reads roughly 6% of the bytes. The rest of the piece is the arithmetic of what that buys you on real benchmarks, and a taxonomy that untangles terms that usually get blurred (storage layout vs. database engine; file format vs. table format; columnar OLAP vs. wide-column stores).

---

## The five mechanisms, in execution order

The guide's model of why columnar wins is not one trick but five multipliers:

- **Column pruning** — name your columns; the engine never reads the other 94%.
- **Data skipping** — zone maps (min/max), Bloom filters, sparse indexes prune whole blocks without decompressing them. On their taxi file, `WHERE trip_distance > 200` skips 39 of 92 row groups.
- **Compression** — same-typed values encode well: dictionary, RLE, delta, frame-of-reference, FSST. 1.09 GB of CSV became 164 MB of Parquet (6.8×).
- **Vectorized execution** — batches of 1,024–4,096 values with SIMD process ~10× faster per cycle.
- **Late materialisation** — don't stitch columns back into rows until after filters have discarded most of them.

The honest framing is worth quoting: each mechanism multiplies the others, but the headline multiples should not all be believed.

> "Treat 160× as a format trick, 22–60× as the working range."

That's `count(*)` answered from Parquet row-group metadata. Most vendor benchmarks would have led with 160×; this one quarantines it. Similarly, on an unsorted taxi file, *no* row group qualifies for skipping under a distance filter — sort order matters, and the guide says so rather than letting zone maps look free.

## Layout vs. engine, file vs. table format

The conceptual scaffolding is the durable part of the piece:

- **Columnar storage** is the layout. **A columnar database** is a query engine built around it (vectorized execution, zone maps, predicate pushdown, SQL). Parquet is columnar storage in a file; DuckDB is a columnar database.
- **Parquet** is how data rests on disk; **Arrow** is how it moves and computes in memory; **DuckDB's internal format** is columnar but mutable, with ACID inside one file.
- **Iceberg, Delta Lake, Hudi, DuckLake** are table formats *on top of* Parquet — snapshots, schema evolution, ACID — "not a third storage layout."
- **Cassandra, HBase, Bigtable, ScyllaDB** are wide-column stores: row-oriented under the hood, despite the name. This is the single most common confusion in the space and the FAQ addresses it head-on.

## When not to use it

The anti-advice is unusually concrete: OLTP and single-row writes (a one-row update touches many column segments), `SELECT *` by primary key, tiny tables where per-file metadata can lose to SQLite, trickle ingest that fragments the layout, and named per-engine limits (Snowflake's 60-second warehouse-resume minimum, MotherDuck's single-node memory ceiling). "A columnar warehouse does not replace the application database. Keep both."

The guide closes with a claim aimed at this wiki's readership: columnar storage is "the ideal substrate for agent and LLM analytics" — sub-second structured context for agents, embeddings stored as nested Parquet lists or in Lance, FP16/FP8 halving the bytes.

## Themes

#concept #tool #pattern

## Opinion

This is vendor content (MotherDuck selling serverless DuckDB) that behaves like a textbook, and that's rarer than it should be. The self-limiting moves — the 160× disclaimer, the unsorted-file confession that data skipping did nothing, the "do not use us for OLTP" section, the table where MotherDuck's own memory ceiling is listed as a key limit — buy credibility that the ClickBench numbers then spend. The five-mechanism model is the right pedagogy: it explains why columnar wins multiplicatively and predicts exactly when it loses (narrow queries on tiny tables, high-cardinality unsorted strings, whole-row reads).

Weaknesses: the engine comparison table is a mile wide and an inch deep (Druid and Pinot reduced to one limit each), and the closing agent-analytics paragraph is a nod to the future rather than an argument. But as orientation for anyone choosing a storage layout, this is the best single-page treatment in the wiki's collection — the taxonomy alone (Parquet/Arrow/DuckDB-internal/table formats/wide-column) resolves five confusions in one table.

## Relations

- This strengthens [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]] with a clean statement of the Parquet-rests/Arrow-moves division of labour that McKinney's ecosystem work built.
- It nuances [[MotherDuck Data Warehousing Overview]] by naming MotherDuck's own scaling limit (one node per query, memory ceiling where MPP clusters spill) inside a fair engine table.
- It gives concrete mechanism behind [[DuckDB ADBC Extension]]'s subject: why DuckDB's columnar internal format matters when other tools connect to it.
- It complicates any reading of [[Drilldown Dashboards from a Single Parquet File]] by showing exactly which query shapes a single Parquet file serves well — and which it doesn't.

---
*Sources: [[raw/columnar-storage-guide]], [[summary/columnar-storage-guide]]*
*Last updated: 2026-10-12*
