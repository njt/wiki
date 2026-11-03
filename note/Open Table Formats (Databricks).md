# Open Table Formats (Databricks)

Databricks' comprehensive explainer of open table formats — the metadata layers that turn loose Parquet/ORC files in object storage into governed, ACID-compliant, time-travelable tables. It compares Apache Iceberg, Delta Lake, and Apache Hudi by design lineage, walks through each format's mechanics, and ends with operational guidance on maintenance, concurrency, and format selection. Being a Databricks piece, it quietly argues Delta Lake's case throughout.

---

## What it covers

- **Lake → lakehouse**: object storage scaled cheap but never had a notion of table, schema, or transaction; the open table format is the bridge, "turning loose files in object storage into governed, queryable tables without copying data into a proprietary warehouse."
- **Three formats, three origins**: Iceberg (Netflix, multi-engine analytics, manifest-tree metadata, partition evolution), Delta Lake (Databricks, Spark-native, append-only transaction log with checkpoints), Hudi (Uber, record-level indexes built for frequent upserts and CDC).
- **File format ≠ table format**: Parquet defines how one file is organized; the table format defines how many files become one consistent table. The piece claims this distinction "clears up most enterprise confusion."
- **Convergence**: deletion vectors, row lineage, Variant, and catalog-coordinated commits are spreading across formats — Delta's catalog commits align it with Iceberg's catalog-as-source-of-truth model, and a unified Delta/Iceberg metadata structure is in preview.

## Key quotes

> Object storage was never built to guarantee consistency, though: it has no native concept of a table, schema, or transaction.

The whole category exists because of this one gap. Every format is a different answer to the same question: who is the source of truth for "what is the current state of this table"?

> Because Delta Lake's original commit model depended on file system access, third-party engines outside Apache Spark had to reach tables through static file paths rather than a governing catalog — leaving those accesses ungoverned and able to silently break schema relationships.

A remarkably candid admission of Delta Lake's original architectural weakness, framed as the motivation for catalog commits. The framing is vendor-flattering — "aligning Delta Lake with the catalog-oriented approach Iceberg has used from the start" — but the concession is real.

> Because every write produces a new snapshot with its own manifest list, an actively written Iceberg table can accumulate thousands of small manifest and data files if left unmanaged.

The trade-off stated plainly: the snapshot model that buys time travel also buys metadata sprawl, which erodes exactly the query performance the metadata tree was built to deliver.

> Engine compatibility should be evaluated against the query engines already in production, not just a feature list — a technically superior format lacking a mature connector for a team's primary engine adds more risk than it removes.

The best advice in the piece, and the one least aligned with vendor interests, since it devalues format feature comparisons in favour of boring fit.

## Themes

#concept #tool #pattern

## Analysis

This is a solid technical explainer with a vendor accent. The mechanics — metadata trees, transaction logs, two-tier data skipping, optimistic concurrency — are explained accurately and pedagogically, and the file-format/table-format distinction is genuinely clarifying. But the selection is Databricks-shaped: Hudi gets the thinnest treatment, Iceberg's manifest maintenance burden is dwelt on, and the piece ends on Unity Catalog and Databricks' "open, format-agnostic lakehouse storage" — the word *open* doing a lot of work for a company whose format competes with the community standard.

The most interesting thread is convergence. The piece argues Iceberg, Delta, and Hudi are growing into each other: Delta adopting catalog coordination, Iceberg adopting deletion vectors and row lineage, both adopting Variant. If formats are converging, the differentiator shifts from protocol to the governance layer — which is exactly where Databricks wants the competition. Read the conclusion with that in mind: "the practical next step" happens to route through their catalog.

The operational guidance is the durable part: compaction as scheduled runbook not ad hoc task, vacuum windows sized to the longest-running query, monitoring file counts and commit failure rates before they surface as slow queries, and rehearsing snapshot rollback before you need it. This is standard SRE discipline applied to metadata, and it transfers regardless of format.

## Related pages

This source is the canonical background for [[Iceberg Table Cleanup]] — the cleanup guide's "append-only means everything grows without bound" premise is exactly the manifest-sprawl problem this piece describes, and its expire → compact → manifest sequence is the operational answer to this piece's "scheduled compaction" remedy.

It also fills in the metadata-layer theory behind [[What Is Columnar Storage (MotherDuck)]] — that guide's Parquet/Arrow/table-format taxonomy is expanded here into how the table formats themselves cache file-level statistics for two-tier data skipping.

And it sharpens the contrast with [[DuckLake — SQL as a Lakehouse Format]]: DuckLake's whole pitch is that file-based metadata (manifests, logs, sprawl, compaction) is the wrong design, whereas this piece treats that metadata layer as the category's crowning achievement — the two sources are arguing opposite sides of the same architecture question.

---
*Sources: [[raw/open-table-formats]], [[summary/open-table-formats]]*
*Last updated: 2026-11-03*
