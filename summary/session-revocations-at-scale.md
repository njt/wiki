---
url: https://www.canva.dev/blog/engineering/session-revocations-at-scale/
title: "Session revocations at scale"
author: Llew Vallis
date_fetched: 2026-08-01
date_published: 2026-07-22
topics:
  - software-engineering-craft
---

Canva manages sessions for hundreds of millions of users, with every backend request requiring user identity — answered hundreds of thousands of times per second. They use encrypted browser cookies containing identity, permissions, and roles so gateways can trust cookie details without a networked datastore lookup per request. When users are logged out or permissions change, revocations must propagate near real time.

Gateways each maintain an in-memory revocation cache. On startup, hundreds of pods each pulled over a million revocations from MySQL, causing a coordinated stampede during deployments. The team replaced this with S3-hosted chunks: 30-minute sliding windows of revocations stored as flat, sorted binary arrays. Each revocation packs a principal and login-timestamp cutoff into 16 bytes via bit twiddling, enabling binary search over downloaded bytes with no transformation. This cut memory footprint by 87.5% versus the prior multi-Java-object representation.

An async worker scans the database, reads the latest S3 chunk (or creates a new one), inserts revocations into the sorted array, and re-uploads. Optimistic concurrency via conditional PUT requests guarantees append-only correctness. ZooKeeper leader election reduces conflicts but correctness doesn't depend on it. Throughput exceeds 2,000 revocations per second — more than needed for the foreseeable future.

Gateways download only chunks from the last 12 hours (all tokens refresh within this window), polling the latest chunks via conditional GET requests a few times per minute. Even a million revocations in a 30-minute window is only 16 MB — negligible versus gateway proxy traffic.

Key outcomes: faster deployments, reduced read replicas to two, and database load that now scales predictably with write throughput rather than exploding with fleet size. The team found that in practice, the worker bottlenecked on network latency, not on sorting dense arrays — a gap between theoretical and actual scaling limits.

---
*Source: [[raw/session-revocations-at-scale]]*
*Last updated: 2026-08-01*
