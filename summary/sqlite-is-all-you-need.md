---
url: https://www.dbpro.app/blog/sqlite-is-all-you-need
title: "SQLite Is All You Need"
author: Jay
date_fetched: 2026-07-18
date_published: 2026-07-14
---

Jay builds Chirp, a social network with 50,000 users and 1 million posts, running entirely on a single 343MB SQLite file — to test whether SQLite can serve production workloads. Every write is a durable, committed transaction. The backend is one Node process talking directly to the file; there is no database server, no connection pool, and no network round-trips.

Production configuration comes down to five pragmas: WAL journal mode, `synchronous = NORMAL`, a 5-second busy timeout, foreign keys enforced, and a 64MB cache. End-to-end HTTP benchmarks on an Apple M1 reach 51,427 req/s for point reads and 3,543 req/s for the heavy timeline query (a mixed read/write workload hits 3,654 req/s). Raw query benchmarks show 232,011 reads/s for simple point reads and 23,459 writes/s for individual inserted posts.

The WAL journal mode is the key enabler. With a writer sustaining 1,000 writes/s, WAL delivers 5.6× the read throughput of the rollback journal (2,792 reads/s vs. 497), with p99 latency of 4.40ms vs. 133ms, and zero `SQLITE_BUSY` errors. The tradeoff is that active writes still suppress read throughput significantly — a 6.3× drop from idle — because there is a single global writer lock.

Jay is explicit about SQLite's limits: a single writer serializes all writes, there is no built-in failover, and recovery time is bounded by file-restore speed. Postgres becomes the right call when many writers contend on the same rows, when read replicas or automatic failover are needed, or when a real analytics engine is required. "We might scale one day" is not a good reason. Cloud estimates suggest a $12/mo droplet can serve roughly 110 million timeline requests a day. The advice: buy single-core speed and enough RAM to hold the database on local NVMe — never network storage.

The core argument is that premature infrastructure complexity is a tax paid for traffic you don't have. If SQLite ever becomes the bottleneck, you will have revenue, a team, and a clear signal about what to fix. Backups are a one-liner shell command; continuous replication works via Litestream. Operational simplicity — nothing to provision, upgrade, or monitor — is the thesis.
