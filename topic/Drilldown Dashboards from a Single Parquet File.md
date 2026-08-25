# Drilldown Dashboards from a Single Parquet File

Hamilton Ulmer's existence proof that a customer-facing analytics dashboard needs neither a database nor a query engine: precompute the dashboard's bounded questions as GROUPING SETS, stack them into one sorted Parquet data cube on R2, and let an 18KB browser reader fetch only the row groups it needs via HTTP range requests. A 34-million-row NYC 311 dataset collapses into a 40MB cube where clicking "NYPD" reads 260KB instead of the file.

---

## Key Quotes

> "With that, you can serve a real drilldown dashboard with neither a database nor a query engine."

The thesis in one sentence. The article's whole energy is the heresy of it — bypassing not just the server but the engine, on the reasoning that a dashboard answers a *fixed* set of questions, so every `GROUP BY` can be precomputed ahead of time.

> "In analytics, when all you have is object storage, everything looks like a range request."

Ulmer's riff on the hammer/nail proverb, aimed at the modern data stack. This is the conceptual bridge to [[Random Access Parquet (RAP)]] — both treat object storage as a directly addressable serving layer rather than a batch-only lake.

> "The real complexity is almost entirely offloaded to the data cube layout."

The load-bearing move. Latency doesn't come from the reader; it comes from how the rows are sorted. Each grouping set is sorted by the columns its queries filter on, so matching rows form a contiguous stretch that Parquet's min/max statistics can isolate. Sorting is the database work.

> "That is why clicking NYPD in the agency leaderboard reads about 260KB out of the 40MB file rather than the whole file."

The concrete number that sells it. The same logic as [[Replace Athena with DuckDB (Lambda)]]'s column pruning, taken further: not just skipping columns, but skipping *row groups* by their statistics.

> "My favorite part is that this approach shifts the complexity 'left' all the way to the data pipeline."

Ulmer's candid bias (he works at MotherDuck) is also his sharpest point: when you *do* buy a real analytical database, the pipeline is where the money goes anyway. So this architecture doesn't add cost — it relocates cost to the thing you were already paying for.

> "An 18kb javascript reader and a byte layout that does the database work for you. What a world!"

The closer, and the true innovation is in the second clause. Hyparquet is just the enabler; the *byte layout* is the database.

> "One file per customer also makes auth refreshingly boring."

A signed URL per customer file, or a tiny Worker checking the session. This is the same "the file is the isolation boundary" move as [[celld]]'s SQLite-per-cell, applied to multi-tenant dashboards.

## Key Themes

#concept #tool #pattern

- **Precompute, don't query.** The dashboard's questions are bounded, so materialize every `GROUP BY` as a grouping set ahead of time. This is OLAP's cube idea stripped of the warehouse — a [[Malloy]]-adjacent semantic instinct executed in raw Parquet layout.
- **Sort order is the index.** [[Random Access Parquet (RAP)]] builds an *external* index to find rows fast; this approach needs none, because sorted layout plus built-in row-group min/max metadata does the pruning. Fewer moving parts, but only for aggregate/grouped questions, not arbitrary point lookups.
- **Range requests over a static file** is a mature pattern: PMTiles (Hilbert-curve layout), SQLite-over-HTTP (B-trees over ranges), and now Parquet cubes. The only requirement is a host that serves byte ranges — R2, S3, GitHub Pages, nginx.
- **The pipeline is the product.** Economics shift cost left to the rebuild step (R2 writes at 12.5× reads, one write per customer per rebuild). Iceberg snapshot diffs tell you which customers have new data, so you only rebuild cubes with activity.
- **Aggregation type is the boundary.** Distributive and algebraic aggregations (sum, count, max, average) decompose across row groups; holistic ones (median, distinct count) need the full distribution and are left "as an exercise to the reader."

## Critical Analysis

**This is RAP for the aggregate case, and it's weaker and stronger for exactly that reason.** [[Random Access Parquet (RAP)]] serves *arbitrary point queries* via an external index — powerful but you must build and keep that index consistent with the lake. Ulmer's cube serves *only precomputed aggregations*, which is a severe restriction — but it costs nothing extra: the footer min/max statistics ship free inside every Parquet file. For the narrow but common case of a customer dashboard over a bounded set of charts, free pruning beats a maintained index.

**The two conditions are doing quiet work.** "The combinatorics of your charts and filters have to stay small, and your pipeline has to rebuild each customer's file fast enough." Both hold for usage/billing pages; both break for anything exploratory. This isn't a general query engine, it's a rendering format for a fixed set of questions — and Ulmer is honest that the daily-grain choice made his file 7× larger than weekly, which is fine only because latency doesn't care about cube size.

**The economics claim is real but time-boxed.** The cost table (writes at $4.50/million, egress free on R2, ~$1.35/mo for 10K daily rebuilds) is the most persuasive part, and it's anchored on a recent change — R2's free egress. On S3 the same setup is "only about 20% more expensive" at realistic traffic. The unspoken dependency is that your data already lives in Iceberg; if you'd need to build the lake to get here, the pipeline "you were already paying for" is new spend.

**The cleanest idea isn't the cube — it's one-file-per-customer as an auth and isolation primitive.** Signed URLs and free egress make per-tenant dashboards boring in the way [[SQLite Is All You Need]] makes the single-file database boring: the file is the boundary, so there's no session database, no row-level security, no query engine to secure. That's a genuinely under-explored idea for multi-tenant analytics.

**What it doesn't solve is freshness.** Rebuild-on-schedule means your dashboard lags real time by the rebuild cadence. Ulmer waves at Iceberg diffing to make that cheap, but a 5-minute-cadence dashboard over 10K customers is 86M writes a month — $389, which he admits is "probably cheaper" but is no longer free. The latency sweet spot is coarse-grained, read-mostly usage data, not operational telemetry.

## Connections

- [[Random Access Parquet (RAP)]] — the same "serve interactive queries from Parquet in the lake" thesis; RAP uses an external index for arbitrary lookups, this uses built-in stats + sorted layout for bounded aggregates.
- [[Replace Athena with DuckDB (Lambda)]] — the same HTTP-range-request insight, but pushed to the client: no Lambda, no DuckDB, no engine, just an 18KB reader.
- [[SQLite Is All You Need]] — the shared instinct that a single file plus a thin reader can replace a database server for a real class of workloads; the SQLite-over-HTTP writeup is Ulmer's explicit prior art.
- [[Icebug Format]] — the in-place, query-Parquet-from-object-storage family this belongs to.
- [[Databases and Data]] — the hub for this convergent-storage thread: one format, many access patterns.

---
*Sources: [[raw/customer-dashboards-r2-hyparquet]], [[summary/customer-dashboards-r2-hyparquet]]*
*Last updated: 2026-08-25*
