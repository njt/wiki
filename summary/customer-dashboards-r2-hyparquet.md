---
url: https://www.hamiltonulmer.com/customer-dashboards-r2-hyparquet/
title: Fast drilldown dashboards from a single Parquet file
author: Hamilton Ulmer
date_fetched: 2026-08-25
date_published: 2026-08-21
topics:
  - databases-and-data
---

Hamilton Ulmer (MotherDuck) tests whether a customer-facing analytics dashboard can be served from a single Parquet file in object storage — with no database and no query engine.

The recipe: roll the dashboard's bounded set of questions (requests per day, per-agency totals, leaderboards) into precomputed **GROUPING SETS** — one small table per grouping — and stack them all into one sorted Parquet "data cube" on R2. Two Parquet features do the serving work: **row groups** (a few tens of thousands of rows each) and the footer's **min/max metadata** per column. The browser reads the footer once, then uses min/max statistics to fetch only the row groups that could match a filter, aggregating the rows locally with **Hyparquet** — an 18KB JavaScript Parquet reader — instead of a multi-megabyte DuckDB-Wasm plus a worker.

The demo rolls 34 million NYC 311 rows into a 40MB cube; clicking NYPD reads about 260KB of it. The trick works only because the rows of each grouping set are sorted by the columns its queries filter on, so matching rows form a contiguous stretch the min/max stats can isolate. The economics favor it: R2 writes cost $4.50/million (12.5× reads) and one write per customer per rebuild, so a 10K-customer dashboard rebuilt daily costs a few dollars a month in storage and writes, with egress free on R2. Auth becomes a signed URL per customer file.

The real cost shifts to the data pipeline — a `GROUP BY GROUPING SETS` statement in DuckDB produces each per-customer cube — and the approach only handles distributive/algebraic aggregations, not holistic ones. Prior art: PMTiles' Hilbert-curve layout and the SQLite-over-HTTP writeup.
