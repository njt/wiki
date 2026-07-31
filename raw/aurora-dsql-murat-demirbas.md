---
url: https://muratbuffalo.blogspot.com/2026/07/aurora-dsql-scalable-multi-region-oltp.html
title: "Aurora DSQL: Scalable, Multi-Region OLTP"
author: Murat Demirbas
date_fetched: 2026-08-01
date_published: 2026-07-23
---

# Aurora DSQL: Scalable, Multi-Region OLTP

**Author:** Murat Demirbas (blog: *Metadata* — "On distributed systems broadly defined and other curiosities")
**Date:** July 23, 2026

## Overview

The author notes that the Aurora DSQL paper (arXiv:2607.13276) finally shipped, and reads it as an insider: he spent 2022–23 working with the AWS team that designed and built it. Because he already knew the architecture, the paper felt dry to him. His one-sentence distillation of the design:

> "We took a traditional monolithic database and blew out every single component into an independent, horizontally scalable service."

## Exploding the Monolith

DSQL decomposes the database into five specialized services:

- **Query Processors (QP):** Stateless VMs running a custom PostgreSQL engine to parse queries and buffer writes locally.
- **Storage Nodes:** Sharded nodes holding data, using MVCC to serve historical data to QPs instantly.
- **Adjudicators:** The conflict-resolution layer that determines whether a transaction handed off by a QP is safe to commit.
- **Journals:** A highly available replication log that durably saves transactions across zones/regions — providing short-term durability before data reaches storage nodes.
- **Crossbars:** The routing layer that reads updates from Journals and pushes changes to the right Storage Nodes.

## The Big Architectural Bets

1. **The Synchronized Clock Bet:** To get coordination-free reads, DSQL relies entirely on highly synchronized physical clocks — AWS TimeSync. A QP checks its local clock and asks storage for data from that exact microsecond.
2. **No Pessimistic Locking:** OCC would normally cause high abort rates, but DSQL pairs it with MVCC under Snapshot Isolation. Since readers look at a past snapshot, read-write conflicts are impossible. For write-skew, customers are told to use `FOR UPDATE` and design schemas that force write-write conflicts for business-logic violations.
3. **Eventual Consistency is Dead:** DSQL provides linearizability, on the argument that developers "simply cannot write correct business logic on eventually consistent systems." The author clarifies that linearizability (single-object real-time guarantees) and snapshot isolation (multi-object transaction visibility) control different things; DSQL offers a consistent snapshot for transactions plus strict real-time ordering for individual key operations.
4. **Forcing Guardrails:** Transactions are hard-capped at 3,000 rows and 10 MiB. The paper cites Little's Law: smaller transactions buy highly predictable, stable tail latency.
5. **Linearized 2PC:** Traditional 2PC over wide-area networks costs 2 RTTs. DSQL uses a Warp-inspired trick where adjudicators vote, but only the leader writes the final commit to its single Journal — avoiding coordination across multiple logs.

## The Payoff

- **Independent Scalability:** Compute, commit logic, and storage scale separately. Add storage nodes for read capacity; spin up more QPs for connection spikes; adjudicator-range placement can even be tuned to access patterns.
- **0-RTT Consistent Reads:** The QP assigns a local timestamp and storage handles the rest — zero leader coordination. This matters because OLTP workloads are read-heavy, and most writes (e.g., UPDATEs, INSERTs with unique indexes) are reads first.
- **1-RTT (or 1.5 RTT) Commits:** Whether writing one row or a hundred, coordination happens once at commit time.
- **No "Slow Lock Holder" Problem:** With no pessimistic locks, "a developer going to lunch with an open transaction terminal" cannot take down the database. Readers never block writers, and writers never block readers.

## Tradeoffs and Shortcomings

The author is candid about downsides:

- The long read-modify-commit window makes DSQL prone to write-write conflicts on hot keys, limiting how many back-to-back operations can hit a single row.
- Under heavy contention, transactions abort rather than queue — unlike a traditional database.
- Cross-region writes to the same key trigger OCC aborts, but the conflict is only detected at commit time, so you pay the WAN latency penalty before learning you must retry.
- The paper lacks extensive quantitative evaluation and adoption data, though it does include a candid Lessons Learned section on friction with foreign key constraints and high-locality sequences.

## Building at Scale

The author's final, counterintuitive takeaway: building a novel global production database felt like less effort than it should have. He credits a great upfront design and a deliberate choice to reuse existing pieces:

- PostgreSQL's engine was used for SQL parsing, execution, and the client protocol — while its local storage and transaction processing layers were discarded.
- The team reused AWS's existing internal Journal service rather than building a new replication log.
- Hard-earned lessons came from prior AWS database projects, including JournalDB and QLDB.

He also praises team dynamics, calling Marc Brooker "technically brilliant" and a masterful leader of a talented principal-engineer team, with enjoyable weekly whiteboard sessions. The author caveats that he wasn't with the team for the final year of the project.

---

*Labels: databases, distributed transactions, distSQL, paper-review*
