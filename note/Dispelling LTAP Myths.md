# Dispelling LTAP Myths

Databricks rebuts four misreadings of LTAP — that it's CDC, that two formats mean two copies, that analytics is always stale, and that parallel WAL decoding is an easier path — with the central claim that the lake is the database's *durable tier*, not a second database.

---

The post follows up on Reynold Xin's "From monolith to Lakebase to LTAP" explainer, addressing the commentary that read LTAP through designs people already know. Its unifying diagnosis: **all four myths come from treating every materialization of the data as a second database.**

## The tier/replica test

The load-bearing distinction is between a *tier* and a *replica*:

> So the test is simple: can this layer be queried, or failed over to, as its own database? If yes, it is a replica. If no, it is a tier.

By that test Postgres was always tiered — WAL, buffer cache, and indexes all overlap the heap's bytes, yet "nobody calls an index a replica of the table, because it has no catalog or timeline of its own." LTAP just moves that hierarchy onto object storage and lets a lakehouse engine read the bottom tier. CDC, by contrast, genuinely creates a second database: an independently queryable upserted table the source never reads back, prone to type drift, missing DDL, dropped replication slots, and opt-in table selection.

## Two lossless encodings, one table

The two-format objection (heap for lookups, Parquet for scans) rests on the assumption that one copy implies one encoding. Because both encodings are lossless, the columnar copy can be the durable store and heap pages become a cache of it — structurally identical to how Postgres already serves a heap and a covering index as two access paths over one table. An honest concession: the columnar files are heap-ordered, so warehouse-style clustering must still be derived like silver tables, just without the CDC pipeline.

## Consistency by LSN, not by pipeline luck

> The lake does not need to contain the latest commit for that commit to be visible. It needs to contain almost everything, plus a way to fetch pages that got committed after the last materialization.

Analytical queries read every table at a *single LSN* — a shared watermark, the same cutoff Postgres uses for REPEATABLE READ. This quietly fixes a real CDC defect the post names without dwelling on: CDC applies table-by-table, so joins mix points in time. Small tables are reframed as a metadata cost, not a freshness problem — they stay in the page cache and skip Iceberg commits.

## Why parallel WAL decoding doesn't work

> WAL records are ordered because later ones assume earlier ones. Visibility lives in pg_xact, the proc array, and the catalog at that LSN, not in the tuple.

You cannot partition a WAL byte range across workers and emit correct current rows. What LTAP parallelizes is *pages*: WAL summaries (Postgres 17+) or pageserver LSM trees identify dirty pages, each an independent transcode job, committed as one Iceberg snapshot. The snapshot is the current pages, not a replay of every record that dirtied them. The post is fair to logical replication for its intended jobs — migrations, upgrades, cutovers — and scathing about using it *as storage*.

## Take

This is a vendor post defending its architecture, but the argument earns its keep on rigor rather than marketing: it gives you a precise test (tier vs replica), a precise failure mode of CDC (two catalogs, per-table watermarks), and a precise reason parallel log decoding is hard (visibility state lives outside the tuple). The strongest structural point is the reframing — "second copy" confusion is endemic in data engineering, and the index-is-not-a-replica analogy is the cleanest teaching device for it. The weakest spot is the missing quantification: "a fraction of" WAL volume and unspecified compaction intervals are asserted, not measured, and the post is silent on what happens under heavy write bursts when pageservers become the freshness bottleneck.

## Themes

#concept (tier vs replica) #pattern (one durable copy, many encodings) #tool (Lakebase/LTAP)

## Related

- Strengthens [[Open Table Formats (Databricks)]]: this is the operational-architecture sequel — where Iceberg/Delta sit as the durable tier under a live OLTP database rather than as analytics targets fed by pipelines.
- Complicates [[DuckLake — SQL as a Lakehouse Format]]: DuckLake solves lake-warehouse consistency by putting metadata in a SQL catalog; Lakebase solves it by making the lake the database's own durable tier — two different answers to the same "no second source of truth" problem.
- Nuances [[What Is Columnar Storage (MotherDuck)]]: the heap-vs-Parquet pair here is a live example of row and columnar layouts coexisting as access paths over one logical table, not two competing storage choices.
- Connects to [[Cloudflare K2 — Serverless Event Streams]]: both treat object storage as the durable substrate that offloads log/replay machinery, with compute layers as ephemeral caches over it.

---
*Sources: [[raw/dispelling-ltap-myths]], [[summary/dispelling-ltap-myths]]*
*Last updated: 2026-11-13*
