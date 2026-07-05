---
url: https://web.stanford.edu/class/cs345d-01/rl/aries.pdf
title: "ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging"
author: "C. Mohan, Don Haderle, Bruce Lindsay, Hamid Pirahesh, Peter Schwarz"
date_fetched: 2026-07-05
date_published: 1992-03
publication: "ACM Transactions on Database Systems, Vol. 17, No. 1, Pages 94-162"
---

## Abstract

ARIES (Algorithm for Recovery and Isolation Exploiting Semantics) is a transaction recovery method supporting partial rollbacks, fine-granularity (record) locking, and recovery using write-ahead logging (WAL). It introduces the paradigm of **repeating history** — redoing all missing updates before undoing loser transactions during restart. ARIES uses a **log sequence number (LSN)** on each page to correlate page state with logged updates. All updates are logged, including those performed during rollbacks (via compensation log records, or CLRs). Chaining CLRs to their predecessors ensures bounded logging during rollbacks even across repeated failures. ARIES supports fuzzy checkpoints, selective and deferred restart, fuzzy image copies, media recovery, and high-concurrency lock modes (increment/decrement) requiring operation logging. It enables parallelism during restart, page-oriented redo, and logical undo. The paper shows why System R's shadow-page paradigms are inappropriate for WAL and compares ARIES to other WAL-based methods (IMS, DB2, NonStop SQL, AS/400). ARIES has been implemented in IBM's OS/2 Extended Edition Database Manager, DB2, Workstation Data Save Facility/VM, Starburst, QuickSilver, and the University of Wisconsin's EXODUS and Gamma.

## Key Technical Contributions

### Repeating History Paradigm
The central insight: during restart after a crash, first **redo** all updates that might have been lost (including those of uncommitted "loser" transactions), then **undo** only the loser transactions. This is simpler and more efficient than selectively redoing only winners.

### LSN (Log Sequence Number)
Every page stores the LSN of the last update applied to it. Every log record has a unique, monotonically increasing LSN. This allows the system to determine, for any log record, whether its effects are already reflected in a page — making both redo and undo idempotent.

### CLRs (Compensation Log Records)
Updates performed during rollbacks are logged as CLRs. CLRs are **redo-only** (undoing a CLR would be incorrect). Each CLR contains an UndoNxtLSN pointer to the predecessor of the log record being compensated, allowing the system to skip already-undone work if a failure interrupts rollback. This guarantees bounded logging even with nested rollbacks and repeated failures.

### Three-Pass Restart Recovery
1. **Analysis Pass**: Scan the log from the last checkpoint forward, rebuilding the dirty page table and transaction table to determine which transactions were active and which pages might be dirty.
2. **Redo Pass**: Reapply all updates from the earliest potentially-dirty page LSN forward, using per-page LSNs to skip already-applied updates.
3. **Undo Pass**: Roll back loser transactions in reverse chronological order, writing CLRs for each undo action. The UndoNxtLSN chain allows precise tracking of rollback progress.

### Physiological Logging
Page-oriented redo (for performance — redo can be applied to a page without understanding the logical operation), but logical undo (for concurrency — undo need not be the exact physical inverse, enabling operations like index-tree structure modifications during undo).

### Additional Features
- **Fuzzy checkpoints**: Don't require quiescing; log the dirty page table and active transactions at a point in time
- **Operation logging**: Supports increment/decrement and other commutative operations via logical logging rather than blind before/after images
- **Media recovery**: Fuzzy image copies + log replay
- **Nested top actions**: For internal operations (B-tree splits, etc.) that shouldn't be rolled back by parent transaction rollback
- **Selective and deferred restart**: Offline objects can be recovered later while online objects are available immediately

## Comparison with Other Methods

**System R (Shadow Paging)**: Shadow pages incur extra I/O for page table updates and force pages back to their original disk locations. Not suitable for WAL. Shadowing makes fine-granularity locking, logical undo, and varying-length objects difficult.

**IMS**: Undoes the same non-CLR updates multiple times if failure interrupts restart. No per-page LSN — can't do page-oriented redo or avoid redundant undos.

**DB2, NonStop SQL, AS/400**: Can undo CLRs multiple times, causing severe problems in production. The UndoNxtLSN chaining in ARIES fixes this.

## Related Pages
- [[Databases and Data]] — Hub page
- [[Distributed Systems]] — Agent orchestration as distributed systems
- [[Software Engineering Craft]] — Foundational engineering principles
- [[Queues Don't Fix Overload]] — Managing back-pressure vs. treating symptoms (analogous to WAL vs. shadow paging)
- [[SDPD — Systems Design Police Department]] — Failure modes in distributed systems
