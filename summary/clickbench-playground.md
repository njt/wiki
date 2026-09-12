---
url: https://clickhouse.com/blog/clickbench-playground
title: ClickBench Playground — How I Host 100 Database Systems for an Interactive Benchmark
author: Alexey Milovidov
site: ClickHouse Blog
date_fetched: 2026-08-06
topics:
  - databases-and-data
---

# ClickBench Playground — How I Host 100 Database Systems for an Interactive Benchmark

Alexey Milovidov (creator of ClickHouse) describes the engineering journey behind [benchmark.clickhouse.com/playground](https://benchmark.clickhouse.com/playground), an interactive platform hosting ~100 database systems — relational, analytical, dataframe, and exotic (BQN, Elastic, Mongo, Pandas, Polars) — each preloaded with 100 million records for live query comparison. Users can run queries, create/drop tables, and race multiple systems head-to-head.

## Origins: ClickBench

The project began as the ClickBench benchmark (2013 for ClickHouse, opened in 2022 to all analytical databases). Each database was a directory of shell scripts: install, load data, run queries, output results. The low-ceremony approach — run on EC2, wrap in Docker when needed, configure obscure JVM versions in scripts — made it the most popular open benchmark for analytical databases.

## The Refactoring Problem

Maintaining ~100 shell scripts that all did repetitive things (download dataset, run queries) slightly differently became unsustainable. Competitors cheated by "forgetting" to flush page caches or excluding optimization work from loading time. To enforce restarts before cold queries, every system needed a common interface (`query`, `stop`, `start`, `check`). Refactoring a hundred shell scripts was "well beyond human capabilities" — Milovidov tried manually, then with AI, and after several attempts succeeded.

## Hosting 100 Databases: The Infrastructure

The core challenge: host ~100 database systems interactively without spending $536K/year on EC2. The solution was a single large bare-metal machine running Firecracker microVMs — "building a cloud inside a cloud."

**CPU**: oversubscribed with fair sharing; a watchdog kills runaway processes.

**Memory**: Firecracker's lazy memory allocation means a VM sees 16 GB but the host only allocates what's used. For systems needing more, swap space on the guest's virtual disk maps to the host's page cache via disabled fsync — making memory elastic. When contention spikes, the host OOM killer resolves it.

**Disk**: The most intricate problem. Snapshots of all systems (dataset + memory image) need ~100 TB. Tricks employed: sparse files (zero-initialized guest memory via `init_on_free=1`), reflinks (XFS copy-on-write to avoid duplication), and finally BtrFS with forced zstd compression — which, after a false start where BtrFS misjudged compressibility from first bytes, fit all hundred systems into 7.5 TB. Starting from a snapshot takes under 5 seconds.

**Network**: Internet access during image creation; stripped during queries. For data-lake systems needing S3 access, a custom proxy filters by SNI hostname allowlist. Docker inside guest VMs required disabling its iptables meddling and loading extra kernel modules (overlay, veth, br_netfilter).

**Security**: VMs reset from snapshot after any query error — so `DROP TABLE hits` or `SYSTEM SHUTDOWN` doesn't poison the playground for the next user.

## What He Learned

With a hundred systems behind a common interface, Milovidov ran experiments: 10× dataset size (one entrant's non-commercial version had a threshold just above the default), 100 queries with JOINs/window functions (some top entrants over-optimized for the default 43), concurrent query QPS (some systems stop working under parallelism), and tiny-to-huge AWS instances (ClickHouse passed 2 GB; many systems failed).

The playground also surfaces query-language diversity: the same query expressed in SQL, Pandas DataFrame, BQN, Elasticsearch JSON, MongoDB aggregation pipeline, and LogsQL.

The post ends on a personal note: Milovidov "loves to collect various database systems" and found a fellow collector who now works with him at ClickHouse.
