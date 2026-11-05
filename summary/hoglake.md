---
url: https://github.com/PostHog/hoglake
title: "Hoglake — a lakehouse catalog as a service"
author: PostHog
date_fetched: 2026-11-06
date_published: 2026 (repository)
topics:
  - databases-and-data
  - developer-tools
---

Hoglake is PostHog's open lakehouse catalog: an open table format where snapshots, schema evolution, data files and statistics live as rows in Postgres while the data lives as parquet in object storage. Unlike DuckLake, which embeds catalog SQL in every client library, hoglake's catalog is a *service* — clients speak a REST contract (the OpenAPI spec is the single source of truth) and never see the metadata database.

The architecture is deliberate Postgres maximalism: foreign keys with `ON DELETE CASCADE`, advisory locks, partial indexes and range deletes are load-bearing design material. Commits are scoped to the write set — one commit is one snapshot and its cost does not grow with catalog size, verified by benchmark guards showing flat commit latency at 10K preseeded snapshots. Writers upload parquet themselves and ship footer statistics; the server never opens a data file on the commit path (a deferred-stats mode hydrates footers asynchronously). Deletes are deletion vectors, one live per data file, growth-monotonic. Row lineage is server-assigned: contiguous row-id ranges, never reused, preserved across compaction rewrites.

The server is ~65K lines of Kotlin (Ktor, JDBI, Flyway); the commit tail is serialized per catalog by a Postgres advisory lock held for milliseconds, with typed 409/410 refusals (`ddl_since_read_snapshot`, `table_recreated`) that are never retryable — re-read and re-prepare. Maintenance (compaction with an external merge sort, snapshot expiry, physical cleanup, invariant scans) runs as server background jobs under the same locks. Around the core sit a Python client (`pyhoglake`, which owns the parquet writer path), a native Trino connector, a DuckDB extension that speaks the REST contract, a changefeed with consumer offsets, a replication daemon (hedgerow), a Kafka ingester (millrace, SlateDB-staged at-least-once), a benchmark harness, and a React console. Deployed in production as Gigahog in three environments; authentication (ES256 tokens, per-catalog scopes) is opt-in and currently off by default.
