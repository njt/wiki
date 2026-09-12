---
url: https://web.stanford.edu/class/cs345d-01/rl/aries.pdf
title: "ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging"
author: C. Mohan, Don Haderle, Bruce Lindsay, Hamid Pirahesh, Peter Schwarz
date_fetched: 2026-08-09
date_published: 1992-03
topics:
  - databases-and-data
---

The canonical paper on write-ahead logging (WAL) recovery in database systems. ARIES (Algorithm for Recovery and Isolation Exploiting Semantics) is the recovery method that underpins IBM DB2, Starburst, and numerous other industrial-strength transaction processing systems. Published in ACM Transactions on Database Systems (Vol. 17, No. 1, March 1992, pp. 94–162), it remains one of the most-cited papers in the database systems literature.

ARIES guarantees the ACID properties of transactions through three passes during restart recovery. The **analysis pass** scans the log from the last checkpoint to determine which transactions were active and which pages were dirty. The **redo pass** *repeats history* — it reapplies all logged updates, including those of uncommitted transactions, to re-establish the database state as of the moment of failure. The **undo pass** then rolls back loser transactions in reverse chronological order, writing compensation log records (CLRs) for each undone action.

The paper's key innovations: (1) **Repeating history** during redo, rather than selectively redoing only committed transactions, which dramatically simplifies the logic. (2) Placing a **log sequence number (LSN)** on every page to precisely track which logged updates have been applied — this enables fine-granularity (record-level) locking and operation logging (e.g., increment/decrement) rather than just physical before/after images. (3) **Compensation log records (CLRs)** that are redo-only and chained via an `UndoNxtLSN` pointer to the predecessor of the record being compensated, ensuring bounded logging during rollbacks even under repeated failures.

The paper supports partial rollbacks (savepoints), fuzzy checkpoints that don't quiesce the system, page-oriented redo and logical undo for high concurrency in B-tree indexes, steal/no-force buffer management, and recovery independence (one object can be recovered without recovering the entire database). It also explains why System R's shadow-page recovery paradigms are inadequate in the WAL context when fine-granularity locking and flexible storage management are required.

The paper includes a detailed survey and critique of the recovery methods used by contemporary commercial systems (DB2, IMS, AS/400, Encompass, NonStop SQL), highlighting problematic design choices such as compensating compensations and unnecessary undos. A simulation study confirmed ARIES imposes negligible overhead on normal transaction processing.

ARIES has been implemented in IBM's OS/2 Extended Edition Database Manager, DB2, Starburst, QuickSilver, and the University of Wisconsin's EXODUS and Gamma database machine. Its extensions include ARIES/IM and ARIES/LHS for B-tree and hash-based indexes, ARIES/KVL for nested transactions, and adaptations for shared-disk (data sharing) environments.
