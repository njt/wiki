---
url: https://motherduck.com/learn/columnar-storage-guide/
date_fetched: 2026-10-12
---

# What Is Columnar Storage? Column vs Row Databases Explained

13 min read · Last updated BYColumnar storage is a disk layout that stores every value of one column together, instead of storing each row as a contiguous record. A query touching 3 of 50 columns reads roughly 6% of the bytes. DuckDB, ClickHouse, Snowflake, and BigQuery are columnar; PostgreSQL and MySQL are row stores. On ClickBench's 99.9-million-row benchmark, the same data takes 106 GB in PostgreSQL and 94 GB in MySQL but 20.5 GB in DuckDB, and the median query drops from over four minutes to 0.35 seconds — identical hardware, stock configs.

## Key takeaways

- Columnar storage groups values by column. Row storage groups values by record.
- A query on 3 of 50 columns reads about 6% of the bytes, plus compression.
- Columnar engines win at OLAP (scans, aggregates, filters). Row engines win at OLTP (single-row reads and writes).
- Apache Parquet is the standard columnar *file*format. A columnar*database*(DuckDB, ClickHouse, Snowflake) has its own internal layout, indexes, and sometimes updates.
- Do not use a columnar store as your app database. Point lookups, frequent single-row updates, and `SELECT *`on tiny tables are the wrong shape.
- Cassandra, HBase, Bigtable, and ScyllaDB are wide-column stores — row-oriented under the hood, built for high-volume writes and key-based reads — and are not columnar OLAP databases.

## What is columnar storage?

Columnar storage is how the bytes sit on disk. All `customer_id` values are packed together, then all `region` values, then all `amount` values, then all `ts` values. A row store packs `customer_id, amount, ts` for row 1, then the same three fields for row 2.

### Columnar storage vs a columnar database

- **Columnar storage**is the layout. Parquet is columnar storage in a file. A DuckDB table is columnar storage inside the engine.
- **A columnar database**is a query engine built around that layout: vectorized execution, zone maps, predicate pushdown, a SQL parser. DuckDB and ClickHouse are columnar databases. Snowflake and BigQuery are columnar warehouses.

You can have columnar storage without a database (a directory of Parquet files). You cannot have a columnar database without columnar storage.

Copeland and Khoshafian described the idea in 1985. CWI began the MonetDB work in 1993 and open-sourced it in 2004; C-Store arrived in 2005 and became Vertica. Parquet, DuckDB, ClickHouse, Snowflake, and BigQuery made it the default for analytics.

## Columnar vs row storage: what's the difference?

Row storage keeps one record's fields together on disk; columnar storage keeps one field's values together — the difference decides whether a scan reads the whole row or just the columns a query touches.

| Dimension | Row storage | Columnar storage | 
|---|---|---|
| Layout | One record, all fields, then the next record | One field, all rows, then the next field | 
| Best for | OLTP: point reads, inserts, updates | OLAP: scans, `GROUP BY`, filters on a few columns | 
| I/O for `AVG(amount)`on a 20-column table | Reads every field of every row | Reads `amount`(and any filter columns) | 
| I/O for 3 of 50 columns | Reads all 50 fields (100% of the row bytes) | Reads roughly 6% of the bytes | 
| Compression | 1.5–3× typical | 5–10× typical, up to 30× on low-cardinality columns | 
| Concurrency | 1000s of point queries per node | 10s–100s of analytical queries per node | 
| `SELECT *`of one row | Cheap | Rebuilds the row from many columns | 
| Examples | PostgreSQL, MySQL, SQLite | DuckDB, ClickHouse, Snowflake, BigQuery | 

The gap is measurable. On ClickBench (99.9M rows, 43 queries, c6a.4xlarge, stock configs), PostgreSQL stores the dataset in 106 GB with a median query of 4.3 minutes, MySQL in 94 GB at 7.3 minutes, and DuckDB in 20.5 GB at 0.348 seconds — roughly 5× smaller and three orders of magnitude faster on the same machine.

A sales table with 40 columns and a query that averages two of them is a textbook query: the row store eads 40 fields per row, while the column store reads only two.

## Why are columnar databases faster?

Columnar databases are faster on analytics because they read fewer bytes, skip unread blocks, decompress less, operate on batches, and assemble rows late.

Five mechanisms, roughly in execution order:

- **Column pruning.**A query touching 3 of 50 columns reads roughly 6% of the bytes.- `SELECT avg(trip_distance) FROM trips`never opens the other 19 columns.
- **Data skipping.**Unread blocks never get decompressed. Zone maps (min/max), Bloom filters (equality on high-cardinality columns), and a sparse primary index (range on the sort key) do the pruning. In Parquet, those min/max zone maps live in the file footer written at write time. DuckDB's own storage builds the same min/max stats automatically per row group.- `WHERE trip_distance > 10`skips every block whose max is 8. On our unsorted taxi file no row group qualifies — long trips appear throughout, so nothing is skipped.
- **Compression.**Same-type values encode well (dictionary, RLE, delta). On the taxi file, dictionary encoding plus Zstd turned 1.09 GB of CSV into 164 MB of Parquet (6.8×).
- **Vectorized execution.**The engine processes batches of 1,024–4,096 values with SIMD instead of one row at a time. The remaining bytes process roughly 10× faster per cycle.
- **Late materialisation.**The engine defers stitching columns back into rows until after filters run, so it does not assemble unread fields for rows the- `WHERE`clause is about to throw away.

| Mechanism | What it stores | Best for | 
|---|---|---|
| Zone maps | Min/max per block or row group | Range predicates ( `WHERE ts >= …`) | 
| Bloom filters | Probabilistic membership | Equality on high-cardinality columns | 
| Sparse primary index | Sampled sort-key values | Range scans on the sort key | 

Reading 3 of 50 columns saves roughly 94% of I/O, the remaining bytes process roughly 10× faster per cycle, and skipping indexes prune roughly 90% of surviving blocks — each mechanism multiplies the others.

On our CSV-vs-Parquet benchmark (DuckDB v1.5.5, Apple M3 Max, OS-cached, median of 3, 11,198,026 NYC taxi rows): `avg(trip_distance)` was 0.008 s on Parquet-Zstd vs 0.490 s on CSV (~60×). The all-columns top-5 query was still 22×. `count(*)` was ~160× because Parquet answers it from row-group metadata. Treat 160× as a format trick, 22–60× as the working range.

DuckDB can spill sorts and joins to disk and analyze 100 GB on a laptop with 16 GB of RAM. Columns make the working set smaller.

## How does columnar compression work?

Encodings run *before* a general compressor (Snappy, Zstd, Gzip). The engine picks per column.

| Encoding | What it does | Best for | 
|---|---|---|
| Dictionary | Replace repeated values with small integer codes | Low/medium cardinality (country, status, vendor) | 
| Run-length (RLE) | Store value + repeat count | Sorted columns, long runs | 
| Delta | Store differences between neighbors | Timestamps, sequential IDs | 
| Frame of reference (FOR) | Subtract a block minimum, store offsets | Integers in a tight range | 
| FSST | Tokenize common substrings | High-cardinality strings (URLs, names) where a dictionary fails | 

Parquet's spec covers dictionary, RLE, and delta encoding; FOR and FSST are DuckDB's internal storage encodings, not part of the Parquet format itself.

Sort or cluster on a common filter column (usually a date) so RLE and zone maps fire. Unsorted high-cardinality strings compress the least. On the taxi data, dictionary encoding plus Zstd turned 1.09 GB of CSV into 164 MB of Parquet (6.8×); Snappy landed at 5.1×.

## What are the key columnar file formats?

Apache Parquet is the interchange format: row groups, column chunks, pages, a footer of min/max stats. Spark, DuckDB, pandas, and the cloud warehouses all read it. Files are immutable; an update means rewriting the file.

Parquet is how columnar data rests on disk; Arrow is how it moves and computes in memory; engines read Parquet from object storage and decode into Arrow buffers. ORC is the Hadoop-era sibling of Parquet. DuckDB's internal table format is also columnar, but mutable: built for `INSERT` / `UPDATE` / `DELETE` with ACID inside one file, not for shipping datasets between tools.

| Format | Unit | Typical size | Where it lives | Role | 
|---|---|---|---|---|
| Parquet | Row group → column chunk → page | Spark/Trino default 128 MB row groups; DuckDB defaults to ~2 MB on taxi-shaped data with Zstd (122,880 rows) | Object storage, data lakes | On-disk rest format | 
| ORC | Stripe → column stream | Stripes ~200 MB | Hadoop / Hive | On-disk rest format | 
| Arrow | Record batch | Batches of 1,024–65,536 rows | Memory, IPC, Flight | In-memory compute and exchange | 
| DuckDB internal | Row group → fixed-size column block | Fixed-size column blocks in one file | A DuckDB database file | Mutable tables and transactions | 

| Property | Parquet | DuckDB internal | 
|---|---|---|
| Job | Write-once lake files | Active tables inside DuckDB | 
| Structure | File → row groups → column chunks → pages | Row groups → fixed-size column blocks | 
| Updates | Rewrite the file | In-place / delta-style | 
| Use | Interchange, S3, data lakes | Query processing, transactions | 

Iceberg, Delta Lake, Hudi, and DuckLake are table formats on top of Parquet. They are not a third storage layout.

## When should you use a columnar database?

Reach for a columnar database when the workload scans wide tables narrowly, favors reads over single-row writes, cares about storage cost, and benefits from running SQL directly on files.

- The workload is analytical: `SUM`,`AVG`,`GROUP BY`, filters over many rows.
- Tables are wide and queries are narrow.
- Reads matter more than single-row writes.
- Storage cost matters. Columnar files are usually several times smaller than CSV.
- You want SQL on files. DuckDB queries Parquet in place; no load step.

The engines split by architecture, and the architecture decides which limit you hit first. Deep evaluation lives on /learn/best-columnar-databases-2026/.

| Engine | Architecture | Best for | Key limit | Deployment | License | 
|---|---|---|---|---|---|
| SQLite (row-store contrast) | Embedded row store | App-local transactions, point lookups | No column pruning: an analytical scan reads every field of every row | Embedded library | Public domain | 
| DuckDB | Embedded (in-process) columnar | Notebooks, local ETL, SQL on Parquet | Single process on one machine; no shared storage or concurrent writers across users | Library / CLI | MIT | 
| MotherDuck | Serverless DuckDB with Dual Execution | Shared DuckDB from laptop to cloud | Scales up on a single node per query, so there is a memory ceiling where MPP clusters spill and keep going | Cloud + local DuckDB | Commercial | 
| Snowflake | Cloud MPP columnar warehouse | Multi-petabyte batch, high concurrency | Bills a 60-second minimum every time a warehouse resumes; micro-partitions are 50–500 MB uncompressed | SaaS | Commercial | 
| BigQuery | Cloud MPP columnar warehouse | Multi-petabyte batch, GCP ad-hoc SQL | On-demand bills per byte scanned | SaaS | Commercial | 
| ClickHouse | MPP columnar | Event logs, sub-second dashboards | Self-hosted ops; historically weaker ANSI joins, though this has improved | Self-host or Cloud | Apache 2.0 | 
| Redshift | Cloud MPP columnar warehouse | AWS-native BI | Cluster sizing is manual; Serverless bills per second in RPU-hours with a 60-second minimum charge | Managed / Serverless | Commercial | 
| Databricks | Lakehouse SQL (Photon on Delta) | BI on the same lake as ML | Billed in DBUs; you adopt the lakehouse stack | Cloud | Commercial | 
| Druid | Real-time OLAP | High-ingest event streams | Cluster ops; event streams over ad-hoc warehouse SQL | Self-host or Imply | Apache 2.0 | 
| Pinot | Real-time OLAP | Extreme QPS in product analytics | User-facing serving-cluster ops | Self-host or StarTree | Apache 2.0 | 

Logistics company Trunkrs moved operational reporting off Redshift onto MotherDuck for snappier drill-downs in daily meetings.

Implementation that actually matters: sort on ingest by the filter you always use, load in batches of thousands of rows, size Parquet row groups for your writer — Spark and Trino default to 128 MB row groups; DuckDB defaults to 122,880 rows per row group (~2 MB on taxi-shaped data with Zstd) — set `ROW_GROUP_SIZE` if you need larger, expect vectorized batches of 1,024–4,096 values, partition large facts by date, and never `SELECT *` unless you need every column. Columnar starts to pay from roughly ten million rows up — our taxi benchmark ran at 11.2 million and measured 22-60x.

Columnar storage is the ideal substrate for agent and LLM analytics: agents need sub-second structured context, and embeddings sit in the same files as nested Parquet lists or in Lance, where FP16/FP8 instead of FP32 cuts 50–75% of the bytes.

## When should you not use a columnar database?

Skip a columnar database for OLTP-style single-row writes, whole-row lookups, tiny tables, trickle ingest, and workloads that run into a specific engine's own limits.

- **OLTP.**Frequent single-row inserts, updates, deletes. Updating one logical row means touching many column segments. Use Postgres.
- **Write amplification.**Columnar systems write large immutable blocks. A one-row update often rewrites a block.
- `SELECT *`of whole rows, especially one row by primary key.
- **Tiny tables.**A few thousand rows plus columnar metadata can lose to SQLite.
- **Trickle ingest.**One event at a time fragments the layout and wrecks compression. Buffer, then flush a batch.
- **Named engine limits.**SQLite's single-writer lock model makes even light concurrent writes a bottleneck. Snowflake bills a 60-second minimum on every warehouse resume. MotherDuck scales one node per query and hits a memory ceiling where MPP clusters keep going.

A columnar warehouse does not replace the application database. Keep both.

## How do you query columnar data in DuckDB?

Column pruning is just naming the columns you need:

Copy code

```
SELECT avg(trip_distance)
FROM 'trips.parquet'
WHERE passenger_count = 1;
```
DuckDB reads `trip_distance` and `passenger_count`. The other 18 columns are skipped entirely.

Write a columnar file from anything DuckDB can see:

Copy code

```
COPY (
  SELECT * FROM 'trips.csv'
) TO 'trips.parquet' (
  FORMAT PARQUET,
  COMPRESSION ZSTD
);
```
An all-columns query on the same Parquet file is the fair lower bound (22× vs CSV on the taxi data), not the headline case. Name your columns.

### Try this on your own data

Point DuckDB at a Parquet file you already have. `EXPLAIN ANALYZE` shows which columns were scanned; `parquet_metadata()` shows the row-group min/max the skipper used.

Copy code

```
EXPLAIN ANALYZE
SELECT avg(trip_distance)
FROM 'trips.parquet'
WHERE passenger_count = 1;
SELECT
  path_in_schema,
  row_group_id,
  stats_min_value,
  stats_max_value,
  row_group_num_rows
FROM parquet_metadata('trips.parquet')
WHERE path_in_schema IN ('trip_distance', 'passenger_count')
ORDER BY row_group_id, path_in_schema;
```
If `Projections` in the plan names only the columns you asked for, pruning worked. On the taxi data, `trip_distance` row-group maxima range from 64 to over 320,000, so `WHERE trip_distance > 200` lets DuckDB skip 39 of the 92 row groups without reading them.

## Is SQL the standard for columnar databases?

Yes. DuckDB, ClickHouse, Snowflake, BigQuery, and Redshift all speak SQL: `SELECT`, `JOIN`, `GROUP BY`, window functions. Dialects differ (ClickHouse is case-sensitive and picky about joins), but the interface is SQL.

Start using MotherDuck now!

## FAQS

Row storage packs every field of a record together. It is the right layout for OLTP: fetch or update one row. Column storage packs every value of a field together. It is the right layout for OLAP: scan or aggregate a few columns across many rows.

Yes. Apache Parquet is a columnar *file* format: column chunks, page encodings, and a footer of min/max stats. It is not a database. Iceberg, Delta Lake, and DuckLake are table formats that usually store their data as Parquet files on object storage.

Yes. DuckDB is an in-process columnar SQL engine. It stores tables in its own columnar format and reads Parquet, CSV, and JSON in place. MotherDuck runs the same engine in the cloud with shared storage and Dual Execution across laptop and cloud.

Postgres is a row store. It is built for transactions, not wide analytical scans: an 8 KB heap page stores every column of each row together. Extensions such as `pg_duckdb` can run DuckDB inside Postgres for OLAP, but vanilla Postgres remains row-oriented end to end.

Skip it for OLTP: frequent single-row writes, point lookups by primary key, and trickle ingest. A single-row update touches many column segments, and a few thousand rows plus per-file metadata can make DuckDB or Snowflake slower than SQLite for a lookup. Keep the application database on Postgres or MySQL; columnar pays off on large, wide, read-mostly tables.

No. Cassandra — like HBase, Bigtable, and ScyllaDB — is a wide-column store: rows are partitioned by key and hold many dynamic columns, but the on-disk layout is row-oriented and tuned for high-volume writes and key-based reads. A columnar OLAP database (DuckDB, ClickHouse, Snowflake, BigQuery) stores each field as a contiguous stream for analytical scans.

Yes. The usual 2026 pattern is Postgres or MySQL for the application database and a columnar engine for analytics. Keep recent operational rows in the row store, then batch them into Parquet, DuckDB, MotherDuck, or a warehouse. Hybrid Postgres extensions can age rows into columnar storage inside one table.

They support `UPDATE` and `DELETE`, but the write is expensive. Immutable files such as Parquet must be rewritten. Engines with mutable internal storage (DuckDB, ClickHouse) mark or rewrite column segments rather than patching one field in place. Frequent single-row updates belong in a row store; batch the changes and merge them.

Parquet is how columnar data rests on disk. Arrow is how it moves and computes in memory. Engines read Parquet from object storage and decode it into Arrow buffers for vectorized execution. DuckDB speaks both: it writes Parquet for interchange and uses Arrow-shaped batches while a query runs.

No. Iceberg, Delta Lake, Hudi, and DuckLake are table formats: they add snapshots, schema evolution, and ACID over files. The data files underneath are usually Parquet. They are not a third storage layout and they do not replace Parquet, ORC, or Arrow.
