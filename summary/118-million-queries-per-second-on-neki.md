---
url: https://planetscale.com/blog/118-million-queries-per-second-on-neki
title: "118 Million Queries Per Second on Neki"
author: PlanetScale (no byline)
date_fetched: 2026-09-13
date_published: 2026-09-12 (approximate — page undated; post says Neki was released "yesterday")
topics:
  - databases-and-data
  - distributed-systems
---

To celebrate Neki's platform preview, PlanetScale benchmarked their new database and reported sustaining **118,538,803 queries per second for 16 minutes** across 512 shards holding **1.22 PiB** of data, with a peak recording of 118,747,267 QPS. The workload was deliberately simple: single-shard point selects fetching one row by primary key, no writes, joins, or cross-shard queries, with each shard's workload fully isolated.

The headline claim is **linear scalability**. The team targeted 200k QPS per shard, then grew the cluster: 5 shards delivered ~1M QPS, 50 shards ~9.9M, and 512 shards 118.5M — ten times the shards, ten times the throughput, twice over. Per-shard throughput held within 0.8% from 5 to 50 shards; at 512 the shards had headroom left, so the load generator was allowed to push each shard to ~231k QPS.

The run's shape: each shard is a Postgres primary on an `r8g.16xlarge`, fronted by 480 Neki routers (one per `8xlarge` instance). p99 latency was 6.06ms at the router and 13.95ms at the client, with 67 errors per second (about one query in 1.8 million), 15.8M read IOPS across the fleet, and over 2 Tb/s of network traffic.

The authors are upfront about what the benchmark does *not* show: shards were primary-only with no replicas, the workload was read-only, and there was no failover during the measured window. A follow-up article on the engineering behind reaching 100M QPS is promised.
