---
url: https://www.databricks.com/blog/open-table-formats
date_fetched: 2026-11-03
---

Open table formats bring ACID transactions, schema evolution, and time travel to data lakes. Compare Apache Iceberg, Delta Lake, and Apache Hudi today.

Open table formats are metadata layers that sit on top of data files in object storage, adding ACID transactions, schema evolution, and time travel to data stored in a data lake. Apache Iceberg, Delta Lake, and Apache Hudi are the three main open table formats in production use today, and each turns a collection of Parquet or ORC files into a table that behaves like a database: readers see consistent results, writers can update and delete rows safely, and every change is tracked so earlier versions remain queryable.

This overview explains how the main open table formats work, how they compare on ACID transaction support and schema evolution, and how they relate to the data lakehouse architecture — drawing on storage-layer innovations like catalog-coordinated transactions, row lineage, and unified metadata to show where Delta Lake and Apache Iceberg are converging.

A data lake is a centralized repository built on low-cost object storage — Amazon S3, Azure Data Lake Storage, or Google Cloud Storage — that holds structured, semi-structured, and unstructured data in raw, native form. Organizations adopted data lakes because object storage scales cheaply and separates storage from compute, letting any query engine read the same data. Object storage was never built to guarantee consistency, though: it has no native concept of a table, schema, or transaction.

A data lakehouse layers table-like structure, governance, and performance onto that raw storage, combining the low cost of a data lake with the reliability of a data warehouse. The bridge between the two is the open table format: it turns loose files in object storage into governed, queryable tables without copying data into a proprietary warehouse.

Before open table formats existed, running analytics on traditional data lakes caused sustained problems: concurrent writers could corrupt data mid-write, updates and deletes meant rewriting entire partitions, and there was no dependable way to know which files represented a table's current, correct state. Open table formats manage metadata for data files in object storage, tracking exactly which files belong to a table — the standardization that let data lakes finally support database-like features such as record-level updates.

Apache Iceberg, originally developed at Netflix and now an Apache Software Foundation project, was designed to make huge, slow-changing tables fast to query and safe to evolve. Apache Iceberg uses a tree structure for efficient metadata management: manifest files and manifest lists track every data file in a table, letting query engines prune irrelevant data before a scan starts. Iceberg tables support schema evolution and partition evolution without rewriting underlying files.

Delta Lake, created by Databricks and released as open source, brought ACID transactions to Apache Spark workloads through a write-ahead transaction log. Delta Lake originated at Databricks and integrates with Spark natively, though it now supports a broad set of engines through independent connectors. Delta Lake tables record every write as an ordered, atomic log entry, giving readers a consistent view even while new data is being written.

Apache Hudi, short for Hadoop Upserts Deletes and Incrementals, is built around fast, frequent record-level updates. Apache Hudi optimizes for frequent updates and streaming data by maintaining indexes that locate the exact file holding a given record, enabling efficient record-level updates without a full table scan. That design makes Hudi a common choice for change-data-capture pipelines and near-real-time ingestion.

Parquet and ORC are columnar file formats, not table formats: they define how individual data files are organized, not how files become a governed table. Iceberg, Delta Lake, and Hudi are all built on Parquet files — Iceberg and Hudi also support ORC — using file-level statistics Parquet stores to prune data before a query engine reads it. That distinction, file format versus table format, clears up most enterprise confusion about where each layer's responsibilities begin.

The three main open table formats now share more capabilities than they differ on, but the table below highlights where design history still shows through.

| Capability | Apache Iceberg | Delta Lake | Apache Hudi | 
|---|---|---|---|
| ACID transactions | Yes, via catalog-coordinated commits | Yes, via transaction log and catalog commits | Yes, via timeline-based commits | 
| Schema evolution | Full — add, drop, rename, reorder columns | Full, including column mapping | Full, schema-on-write and schema-on-read | 
| Partition evolution | Yes, without rewriting existing files | Limited; typically requires redefinition | Yes, via evolving indexing strategies | 
| Origin | Netflix / multi-engine analytics | Databricks / Apache Spark | Uber / streaming ingestion | 
| Update/delete performance | Deletion vectors and row lineage (v3) | Deletion vectors and row tracking | Native record-level indexes | 
| Multi-engine support | Broad — Spark, Trino, Flink, Snowflake | Broad via Delta Kernel and UniForm | Spark, Flink, Presto, Trino | 

An Iceberg table is defined by a metadata tree, not a single file: a metadata file points to a manifest list, which points to manifest files listing the actual data files that make up a snapshot. This layered structure lets a query engine prune irrelevant manifests and data files using stored column statistics without opening a single file, improving query performance on tables with millions of files.

Every write to an Iceberg table creates a new snapshot — an immutable record of which data files existed at that moment — and the metadata tree keeps a history of previous snapshots. This enables time travel: engines can query a table as it existed at a specific snapshot ID or timestamp, supporting auditing, reproducible ML training sets, and rollback to the last stable state after a bad write.

Iceberg decouples a table's physical partitioning from its query patterns through partition evolution, letting teams change how new data is partitioned without rewriting existing files or breaking queries against the old scheme. Hidden partitioning means analysts do not need to reference physical partition columns directly to get partition pruning.

Because every write produces a new snapshot with its own manifest list, an actively written Iceberg table can accumulate thousands of small manifest and data files if left unmanaged. Query engines still have to open and evaluate each relevant manifest file, so manifest sprawl erodes the query performance gains the metadata tree was built to deliver.

The standard remedy is scheduled compaction: a maintenance job that rewrites small data files into fewer, larger ones and consolidates manifests, run nightly or hourly depending on ingestion volume. Pairing compaction with regular snapshot expiration — removing metadata past a retention window — keeps storage and metadata size under control without limiting how far back time travel can reach.

Delta Lake tables store an ordered, append-only transaction log — a sequence of JSON entries recording every add, remove, and metadata change — alongside periodic checkpoint files that summarize the log for faster reads. Historically, the file system itself acted as commit coordinator, meaning any client with file-level access could write to a Delta table directly, without going through a governing catalog.

ACID transactions ensure data consistency during concurrent writes by requiring every writer to check the current log version, generate a new entry, and commit it only if no conflicting change happened in between; if a conflict is detected, the writer retries. ACID compliance prevents data corruption because a reader never sees a Delta Lake table in a partially written state, and ACID transactions enable complex data operations without conflicts across overlapping partitions.

Because Delta Lake's original commit model depended on file system access, third-party engines outside Apache Spark had to reach tables through static file paths rather than a governing catalog — leaving those accesses ungoverned and able to silently break schema relationships. Databricks addressed this with catalog commits, an open standard letting a catalog such as Unity Catalog act as commit coordinator, so every read, write, and discovery request is authorized centrally. Catalog commits are now generally available, aligning Delta Lake with the catalog-oriented approach Iceberg has used from the start and unlocking multi-table transactions.

A Parquet file organizes tabular data by column rather than by row, grouping values from the same column into contiguous blocks called row groups. Columnar storage lets a query engine read only the columns a query references, skipping the rest — a major reason Parquet outperforms row-oriented formats for scan-heavy workloads.

Every row group carries statistics — minimum and maximum values, null counts, and value distributions per column — written into the file's metadata footer. These statistics let an engine determine, without decompressing any data, whether a row group could possibly match a query's filter.

Open table formats extend this principle a level higher: Iceberg's manifest files and Delta Lake's transaction log both cache Parquet-level statistics at the metadata layer, so an engine can skip entire data files before listing them from object storage. This two-tier data skipping is a major contributor to improved query performance on large tables. Databricks has also extended the model with the Variant data type — now part of Parquet, Delta Lake, and Iceberg — storing semi-structured payloads in typed binary form instead of raw JSON, so engines extract nested fields without expensive parsing.

Data versioning is an open table format's ability to retain a record of every previous table state rather than overwriting data in place, and time travel reads any of those previous versions by version number, snapshot ID, or timestamp. Open table formats allow for time travel and versioning of datasets by design, since every write already creates a new, independently addressable snapshot or log entry.

Incremental processing reads only the rows that changed since a table was last processed rather than rescanning an entire dataset — the pattern behind Change Data Capture (CDC), where pipelines consume just the inserts, updates, and deletes applied to a source table. Row lineage and deletion vectors, introduced to Delta Lake and brought to Iceberg through Iceberg v3, made this cheaper: row lineage tracks which rows changed since a table was last scanned, and deletion vectors represent deleted rows as a compact bitmap instead of rewriting data files.

Retaining every historical version indefinitely is expensive, so open table formats pair versioning with retention policies and a vacuum or expire-snapshots operation that removes data no longer referenced inside the retention window. Running vacuum too aggressively can break time travel queries still referencing older versions, so windows are set to match the longest-running query that might need one.

A reasonable cadence for production tables pairs incremental processing on ingestion-matched intervals — often every 5 to 15 minutes for streaming data ingestion — with vacuum and snapshot expiration run daily, outside peak query hours.

Open table formats improve query performance through metadata management organized in layers: manifest files or log entries describe individual data files, snapshots or log versions describe table state at a point in time, and a catalog tracks which version is currently authoritative. Unity Catalog governs more than 17 exabytes of data in open table formats across enterprise deployments today — a sense of how much metadata a modern lakehouse catalog now manages.

As tables grow, manifest files and transaction log checkpoints can themselves slow query planning, so maintenance should include rewriting manifests into fewer, larger files and tuning checkpoint intervals to write frequency. Databricks and the open-source community are developing a unified metadata structure across Delta Lake and Iceberg, expected in preview in the third quarter, combining Delta Lake's fast-write log with Iceberg's fast-read manifest tree.

Metadata caching — keeping recently accessed manifests, checkpoints, or catalog responses in memory — cuts repeated-query latency by avoiding a full metadata tree walk on every request, which matters most for BI workloads issuing many small queries against the same large tables.

Apache Iceberg, Delta Lake, and Apache Hudi all guarantee serializable or snapshot isolation for single-table writes, so concurrent readers always see a complete, consistent version and never a partial write. Where they differ is coordination: Iceberg has always used the catalog as the source of truth, Hudi uses its own timeline service, and Delta Lake relied on file-system atomicity before catalog commits aligned it with the catalog-coordinated model.

All three formats use optimistic concurrency control: rather than locking a table before writing, a writer reads the current version, prepares its change, and commits only if no other writer has committed a conflicting change meanwhile. If a conflict is detected, the transaction fails safely and either retries against the newer version or aborts, never leaving the table inconsistent.

The most effective way to reduce write contention is shrinking the overlap between concurrent writers: partition ingestion jobs so different pipelines write to different partitions, batch small writes into fewer, larger commits, and scope merge operations to only the partitions they touch. Catalog commits help here too, since multi-table transactions let related updates commit together instead of racing as separate writes from multiple processes.

Apache Iceberg tends to fit best where multi-engine read access matters most — organizations querying the same tables from Trino, Snowflake, Flink, and Spark side by side. Delta Lake fits best for Spark-centric pipelines needing multi-table transactions and fine-grained governance. Apache Hudi fits best for high-frequency, record-level upsert workloads such as CDC replication, where its purpose-built indexing outperforms general-purpose merge operations.

Engine compatibility should be evaluated against the query engines already in production, not just a feature list — a technically superior format lacking a mature connector for a team's primary engine adds more risk than it removes. Delta Kernel, an open-source library in Java and Rust, has become a common way engines add Delta Lake support without reimplementing the protocol; it already powers DuckDB and ClickHouse integrations, and the two implementations are converging on a shared Rust core.

Before standardizing on a format, run a proof of concept against a representative, moderately messy production table: measure write latency under concurrent load, confirm target engines read the format natively rather than through a slow connector, and validate that schema evolution and time travel behave as expected.

A catalog is the system of record tracking which tables exist, where their data lives, and which snapshot is authoritative — options include the original Hive Metastore, AWS Glue Data Catalog, and Unity Catalog, which governs Delta Lake and Apache Iceberg tables together under one set of access policies. Delta Lake UniForm goes further, letting a single copy of Delta Lake data be read natively as an Iceberg table without duplicating storage, addressing the format-lock-in concern that pushes teams to delay standardizing.

The clearest way to mitigate vendor lock-in risk is choosing a format with more than one independent engine implementation, and confirming catalog access — not just file access — is portable across platforms a team might need later. Because Iceberg and Delta Lake are open-source, storing data in either does not by itself lock an organization into one processing engine, though governance layers built on top can vary in portability.

Table maintenance should be codified as a runbook, not an ad hoc task: define which tables need compaction and at what file-size threshold, set retention windows for vacuum and snapshot expiration based on how far back jobs query data, and schedule both outside peak query windows.

Monitoring table health should track file count and average file size per table, commit failure rates, and the age of the oldest data file relative to a compaction schedule, alerting when small-file counts or conflict rates spike. These metrics catch metadata bloat and write contention before either becomes visible as slow queries.

Because open table formats support schema evolution without breaking existing queries, schema changes are often applied directly to production tables — but that ease makes it easy to skip testing. Teams should validate changes against representative downstream queries and periodically rehearse a rollback to a prior snapshot, so recovering from a bad change is a practiced procedure, not a first attempt under pressure.

Apache Iceberg, Delta Lake, and Apache Hudi solve the same core problem — bringing ACID transactions, schema evolution, and time travel to data stored in cheap object storage — through metadata designs shaped by where each started: multi-engine analytics for Iceberg, Spark-native pipelines for Delta Lake, high-frequency upserts for Hudi. Parquet remains the common file format underneath all three, and recent innovations — deletion vectors, row lineage, Variant, and catalog-coordinated commits — are converging across formats rather than staying siloed.

The practical next step for most teams is a scoped proof of concept: pick the format matching an existing engine stack, test it against real production data volumes, and confirm the catalog governing it can extend to a second format later. Databricks' open, format-agnostic lakehouse storage lets teams store data once and query it natively as Delta Lake or Apache Iceberg, governed under one catalog, without duplicating data or locking into a single format.

An open table format is an open-source metadata layer that sits on top of data files in object storage and adds database-like features — ACID transactions, schema evolution, and time travel — to a data lake. Apache Iceberg, Delta Lake, and Apache Hudi are the three main open table formats in production use, and each turns a collection of Parquet or ORC files into a table any supported engine can read and write safely.

A file format such as Parquet or ORC defines how an individual data file is compressed and organized on disk, while a table format such as Apache Iceberg or Delta Lake defines how many of those files together form one consistent, queryable table. Open table formats are built on top of file formats, adding the metadata layer — manifests, transaction logs, and catalogs — that file formats alone do not provide.

Yes. Delta Lake UniForm lets a single copy of Delta Lake table data be read natively as an Apache Iceberg table without duplicating storage, and catalogs such as Unity Catalog can govern both formats side by side under one set of access policies. This interoperability lets organizations standardize on shared governance without forcing every engine onto the same table format.

The best open table format depends on the query engines and workload already in production: Apache Iceberg suits multi-engine analytics across tools like Trino and Snowflake, Delta Lake suits Spark-centric pipelines needing multi-table transactions, and Apache Hudi suits high-frequency, record-level upsert workloads such as CDC replication. A proof of concept against real production data confirms the right fit.

Subscribe to our blog and get the latest posts delivered to your inbox.
