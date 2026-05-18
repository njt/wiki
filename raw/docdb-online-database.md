---
url: https://www.infoq.com/presentations/docdb-online-database/
title: "Stripe's DocDB: How Zero-Downtime Data Movement Powers Trillion-Dollar Payment Processing"
author: Jimmy Morzaria (Staff Software Engineer, Stripe)
date_fetched: 2026-05-18
date_published: 2026-04-30
---

# Stripe's DocDB: Presentation Summary

Recorded at QCon San Francisco, published April 30, 2026. Jimmy Morzaria, Staff Software Engineer at Stripe (previously 5+ years at AWS working on Amazon QLDB and Amazon MSK), explains how Stripe's database tier evolved to support 5 million QPS with 5.5 nines of reliability.

## Key Metrics

- $1.4 trillion in payments processed in 2024 (~1.3% of global GDP)
- 5.5 nines reliability target
- 5M+ database queries per second
- Petabytes of financial data across 2,000+ database shards

## What is DocDB?

DocDB is Stripe's internal Database-as-a-Service built atop open-source MongoDB. It is the composition of a database proxy layer, a control plane, a routing metadata service, CDC (Change Data Capture) systems, and a zero-downtime data movement platform.

### Why In-House vs. Off-the-Shelf?

Three reasons drove the build decision:

1. **Security** — "Building DocDB allowed us to bake in security right from the start, specifically through robust enforcement of authorization policies right at the data layer."

2. **Reliability & Performance** — By exposing "a minimal battle-tested set of functions" to engineers, Stripe prevents unintended performance issues. Multi-tenancy with enforced quotas ensures one user's activity doesn't affect another's.

3. **Scale** — Designed for "seamless horizontal scaling with sharding" without compromising reliability or performance.

## Logical Constructs

Two main abstractions for product engineers:
- **Logical database** — A container of data housing related collections
- **Collection** — Specified with a shard key (e.g., `merchant` on a `payment_intents` collection)

Under the hood, keyspace is divided into **chunks** (contiguous key ranges), each mapped to a physical shard via a **chunk map** stored in the routing metadata service.

## Architecture Evolution

| Era | State |
|-----|-------|
| **2011** | Apps connected directly to MongoDB shards; few shards |
| **2017** | Tens of shards; unsharded data remained; ad hoc maintenance scripts became bottleneck. Database proxy layer added for connection pooling, reliability, access control |
| **2020** | Exponential growth (e.g., .ai domain businesses). Vertical scaling hit limits; some shards reached tens of terabytes |
| **Post-2020** | Control plane, routing metadata service, and online data movement built |
| **Today** | 2,000+ shards deployed as MongoDB replica sets across AZs and regions |

## The Zero-Downtime Data Movement Platform

### Design Principles

- **Consistency & Availability** — Data must remain consistent during migration; downtime is unacceptable. Goal: keep the critical phase "shorter than the duration of a planned database primary failover" and within application retry budgets.
- **Performance** — Preserve throughput on shards involved in migration
- **Granularity & Adaptability** — Support arbitrary chunk migrations between any source/target shards with no limits on in-flight migrations

### Migration Steps (6 phases)

1. **Register intent** in the routing metadata service to move a chunk
2. **Bulk import** using a point-in-time snapshot from source to target shards. Breakthrough: sorting data by most common index attributes (leveraging MongoDB's B-tree storage engine) improved write throughput 10x
3. **Async replication** reads the oplog from source shards via CDC (not directly from the MongoDB shard, to avoid impacting throughput). Replication is bidirectional (source ↔ target) for fast rollback capability
4. **Correctness check** — Point-in-time snapshot comparison between source and target
5. **Traffic switch** using version gating — Proxy servers annotate requests with a routing metadata version. The coordinator fences the source shard by bumping its version, causing stale-version requests to be rejected. The route is then updated in the routing metadata service, and proxies pick up the new route. Entire switch takes milliseconds to 2 seconds max. Failed requests succeed on retry.
6. **Complete migration** in the routing metadata service

### Custom MongoDB Fork

Stripe maintains a custom fork of MongoDB with a patch for version gating and tags appended to write-ahead log entries to prevent cyclical replication loops.

## Use Cases Beyond Horizontal Scaling

- **Splitting shards** n-ways for throughput/storage (especially Black Friday / Cyber Monday)
- **Merging** underutilized shards back together
- **Version upgrades** — Migrate from one MongoDB major version to another (skipping intermediate versions). "We've updated our entire fleet of more than 2,000 database shards using the data movement platform." Provides a fast rollback path.
- **Tenancy migration** — Move between multi-tenant and single-tenant infrastructure as product scale demands

## Q&A Highlights

- **Bidirectional replication:** The target shard is a follower, not an active master. Writes from the replication service are tagged so they don't create cyclical loops when replicating back. At any point, only one shard is primary.
- **Idempotency:** "The MongoDB oplog itself gives us the idempotency" — replaying the write-ahead log always reaches the same end state.
- **Routing eventual consistency:** The fencing mechanism on the primary shard ensures stale proxy servers get requests rejected until they update their routing metadata. Some retry optimization exists in both the proxy layer and the client SDK.
- **Throughput:** Roughly 1.5–2 TB per target shard per day for bulk ingestion, with additional replication time depending on write throughput.
- **No MongoDB aggregation pipeline** is used; no `mongos` or config servers — Stripe built their own proxy and routing metadata service.

## Key Takeaways

1. **Reliability is non-negotiable** — Designed front and center; zero-downtime data movement
2. **Invest in strong foundations** — The same migration platform unlocked horizontal scaling, version upgrades, and tenancy migration
3. **Build vs. buy framework** — Build when it drives long-term strategic advantage, requires unique reliability/security/scale controls, or provides better 3–5 year ROI. Buy for undifferentiated capabilities where the vendor meets security and cost requirements.
