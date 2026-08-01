---
url: https://gtowey.blogspot.com/2026/07/why-are-databases-so-hard.html
title: "Why are databases so hard?"
author: gtowey
date_fetched: 2026-08-01
date_published: 2026-07-30
---

A database reliability engineer argues that databases are a perennial source of
outages not because engineers are incompetent, but because the fundamental
tradeoff between correctness and performance/availability is bounded by the
laws of physics — and those laws cannot be negotiated.

The argument builds in three steps. A single isolated database instance gives
perfect ACID compliance and speed, but hardware fails. Adding high availability
requires a standby copy; keeping that copy perfectly in sync adds latency, and
if we treat any node failure as a whole-system failure, we've arguably
*decreased* availability. Geographic distribution for disaster recovery makes
the latency constraint visceral: a New-York-to-Los-Angeles round trip runs
~130 ms in practice, against a theoretical speed-of-light floor of 32 ms.
No optimization will buy even a single order of magnitude here — the physical
limit is baked into the universe.

The practical compromise most companies land on is strong consistency within
one datacenter paired with asynchronous, eventually-consistent replication to
another region. When the primary site fails, some data loss is expected.

The hardest part of the author's role is communicating this reality.
Stakeholders cycle through demands for consistency, speed, availability, and
cost reduction. Running multiple database systems (Redis for ephemeral data,
PostgreSQL for transactions, key-value stores for scale) helps, but shifts the
burden to picking the right store for each dataset and migrating when the first
choice turns out wrong. The article closes with the observation that this
tension is permanent — which, for a database reliability engineer, is at least
job security.
