---
url: https://jvns.ca/blog/2026/07/17/learning-about-running-sqlite/
title: "Learning a few things about running SQLite"
author: Julia Evans
date_fetched: 2026-07-18
date_published: 2026-07-17
---

Julia Evans reflects on running a Django site backed by SQLite — her fourth
SQLite-backed web project — and what she's learned from the rough edges
encountered in practice.

A slow FTS5 full-text search query (5 seconds on a 4,000-row table) was fixed
by running `ANALYZE`, which dropped it to ~0.05 seconds. She suspects a
query-planning issue and muses about eventually learning to read query plans.

DELETE-heavy cleanup jobs would lock the database for over 5 seconds, causing
other workers to hit timeouts and crash the VM. Her workaround: batch deletes
into smaller chunks. This gave her more appreciation for why someone might
reach for Postgres when concurrent writes matter.

She found ORM performance from Django's ORM mostly fine on a small dataset
(~10,000 rows). Backup is handled two ways: restic with `VACUUM INTO` (plagued
by OOM kills), and Litestream (newer, still evaluating). She backs up to AWS S3
but finds credential generation annoying.

Her [[Mess with DNS]] project has run on SQLite since 2022 after migrating from
Postgres — a move she calls a great choice. She closes by noting how long it
takes to learn basic things about the tools you use every day: she'd been using
SQLite for web projects for four years before learning about `ANALYZE`, and
expects to discover another basic feature in another year or two.

---
*Sources: [[raw/learning-about-running-sqlite]]*
*Last updated: 2026-08-01*
