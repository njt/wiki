---
url: https://planetscale.com/blog/118-million-queries-per-second-on-neki
date_fetched: 2026-09-13
---

We released Neki in platform preview yesterday. To celebrate the release, we wanted to test out running 1 million queries per second on Neki. We hit this goal pretty quickly on 5 shards and decided to see how much further we could push it.

This next run ended with 512 shards running 118 million queries per second with 1.22 PiB of data.

## Linear scalability

The benchmark was very simple. A single-shard point select, one row fetched per-query by primary key. No writes, joins, or cross-shard queries. The workload that each shard receives is isolated, in that there are no single queries that span multiple shards.

Our target was to sustain 200k QPS on each shard, and then grow the cluster to increase throughput. Five shards, then fifty, then 512.

| Shards | Routers | Delivered QPS | QPS per shard | 
|---|---|---|---|
| 5 | 12 | 999,624 | 199,925 | 
| 50 | 48 | 9,923,900 | 198,478 | 
| 512 | 480 | 118,538,803 | 231,521 | 

Ten times the shards, ten times the throughput. Then ten times again. From 5 shards to 50 the per-shard rate held within 0.8%. At 512 the shards still had headroom, so we let the load generator use it and each shard settled at 231k QPS instead of 200k.

## 118.5 million QPS

We sustained `118,538,803` QPS for 16 minutes across 512 shards and 1.22 PiB of data. Our largest recording was `118,747,267`.

- 512 shards, each with one Postgres primary each on an `r8g.16xlarge`
- 480 Neki routers, each on its own `8xlarge`instance
- p99 latency of 6.06ms at the router and 13.95ms at the client
- 67 errors per second, about one query in 1.8 million
- 15.8M read IOPS across the fleet
- over 2 Tb per second on the network

Worth being clear about this run: the shards were primary-only with no replicas, the workload is read-only across queries ranging in complexity, and we did not fail over during the measured window.

We will go into details on the engineering effort and interesting challenges we faced along the way to reaching 100 million QPS in a future article.
