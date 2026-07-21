---
url: https://arxiv.org/pdf/2607.13276
title: "Aurora DSQL: Scalable, Multi-Region OLTP"
author: Marc Brooker, Marc Bowes, Mike Hershey, Zak van der Merwe, James Morle, Matthys Strydom
date_fetched: 2026-07-21
date_published: 2026-07-14
---

# Aurora DSQL: Scalable, Multi-Region OLTP

arXiv:2607.13276 [cs.DB] — CC BY 4.0

## Abstract

Aurora DSQL is a serverless SQL database built for cloud-scale transaction processing with multi-region active-active capabilities. Its architecture is disaggregated, separating compute, storage, and transaction coordination into independent, horizontally scalable services. Query processors run in Firecracker MicroVMs and are PostgreSQL-compatible, operating without local state. The system employs multiversion concurrency control with precise timestamps to enable coordination-free reads and optimistic concurrency control for writes. Coordination is deferred to commit time via distributed adjudicators and the Journal replication system. This design minimizes cross-region latency since coordination is needed only during commits, not for individual statements. The system scales from zero to millions of transactions per second while offering strong consistency, ACID transactions, and continuous availability during zone or region failures.

## Authors

Marc Brooker, Marc Bowes, Mike Hershey, Zak van der Merwe, James Morle, and Matthys Strydom (Amazon Web Services)

## Figures

- Customer architecture with 3 availability zones
- Customer architecture with 2 regions
- Architecture overview diagram
- Cross-region architecture with 3 regions
- Multi-region P50 latency for read-write statements
- Benchmark results: EC2 P99 latency by operation
- Read transaction benchmarks
