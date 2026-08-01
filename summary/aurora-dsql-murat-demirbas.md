---
url: https://muratbuffalo.blogspot.com/2026/07/aurora-dsql-scalable-multi-region-oltp.html
title: "Aurora DSQL: Scalable, Multi-Region OLTP"
author: Murat Demirbas
date_fetched: 2026-08-01
date_published: 2026-07-23
---

Murat Demirbas reviews the Aurora DSQL paper (arXiv:2607.13276) from the perspective of an insider who worked on the AWS team that designed and built it during 2022–23. His one-sentence summary: DSQL took a traditional monolithic database and blew every component out into an independent, horizontally scalable service.

The architecture decomposes into five specialized services: stateless Query Processors running a custom PostgreSQL engine, sharded Storage Nodes using MVCC, Adjudicators handling conflict resolution, Journals providing durable replication across zones/regions, and Crossbars routing updates to storage. Key architectural bets include relying entirely on AWS TimeSync synchronized clocks for coordination-free reads, pairing optimistic concurrency control with MVCC snapshot isolation, committing to linearizability rather than eventual consistency, and hard-capping transactions at 3,000 rows / 10 MiB per Little's Law.

The payoff: 0-RTT consistent reads, 1–1.5 RTT commits regardless of row count, independent scalability of compute/commit/storage layers, and elimination of the "slow lock holder" problem (readers never block writers, writers never block readers). Demirbas is candid about downsides — write-write conflicts on hot keys, transactions aborting under heavy contention rather than queueing, cross-region OCC aborts detected only after paying WAN latency, and limited quantitative evaluation in the paper.

His counterintuitive takeaway: building a novel global production database felt like less effort than expected, credited to strong upfront design and strategic reuse of PostgreSQL's engine, AWS's internal Journal service, and hard-won lessons from earlier projects like JournalDB and QLDB.
