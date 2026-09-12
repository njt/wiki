---
url: https://arxiv.org/pdf/2607.13276
title: "Aurora DSQL: Scalable, Multi-Region OLTP"
author: Marc Brooker, Marc Bowes, Mike Hershey, Zak van der Merwe, James Morle, Matthys Strydom
date_fetched: 2026-07-21
date_published: 2026-07-14
topics:
  - databases-and-data
---

AWS paper describing the architecture of Aurora DSQL, a serverless,
PostgreSQL-compatible OLTP database with active-active multi-region
capabilities.

The design is fully disaggregated into three horizontally-scalable layers:
compute (query processors in Firecracker MicroVMs, stateless), transaction
coordination (distributed adjudicators + a Journal replication system), and
storage. Query processors are PostgreSQL-compatible and carry no local state,
so they can be scaled to zero.

The key latency trick is deferring all coordination to commit time.
Multiversion concurrency control with precise timestamps lets reads proceed
coordination-free, while writes use optimistic concurrency control resolved
by adjudicators at commit. This means cross-region coordination is needed
only for commits — not for every statement — keeping p50 read-write latency
low even across regions.

The system offers strong consistency, full ACID transactions, and continuous
availability through zone and region failures, scaling from zero to millions
of transactions per second.
