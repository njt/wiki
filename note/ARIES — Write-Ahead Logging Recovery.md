# ARIES — Write-Ahead Logging Recovery

The 1992 paper that defined how every major database handles crash recovery. C. Mohan and colleagues at IBM introduced ARIES (Algorithm for Recovery and Isolation Exploiting Semantics), a WAL-based recovery method whose architecture still runs inside PostgreSQL, SQL Server, DB2, and InnoDB three decades later. The core insight — "repeat history, then selectively undo" — is so obviously correct in hindsight that it's hard to appreciate how many wrong turns the field took before this paper.

---

## The Central Idea: Repeat History

Classic recovery methods tried to be clever about restart: figure out which transactions won, redo only their updates, and undo the losers. ARIES does something simpler and more robust: **redo everything**, including the updates of transactions that were in-flight at crash time. Only after the database is brought to a physically consistent state (all logged updates applied) does it undo the loser transactions.

This sounds like more work. It's actually less — because it eliminates the need for complex reasoning about which pages might be in what state. Just replay the log, using per-page LSNs to skip updates that are already present. Then undo with precise tracking.

> "We introduce the paradigm of repeating history to redo all missing updates before performing the rollbacks of the loser transactions during restart after a system failure."

This is the paper's single biggest idea, and it's the one that's still with us.

## The Four Building Blocks

### 1. LSNs on Every Page

Every database page stores the Log Sequence Number of the last update applied to it. Every log record gets a monotonically increasing LSN. To decide whether a log record's effects are already on a page: compare the record's LSN to the page's LSN. If the page LSN ≥ record LSN, the update is already there — skip it.

This is deceptively simple and eliminates entire categories of recovery bugs. No timestamps, no version vectors, no guessing. The page tells you exactly where it stands relative to the log.

> "ARIES uses a log sequence number in each page to correlate the state of a page with respect to logged updates of that page."

### 2. Compensation Log Records (CLRs)

When a transaction rolls back, the undo actions themselves must be logged. These are CLRs. Critically, CLRs are **redo-only** — you never undo an undo. Each CLR chains to the predecessor of the log record it compensates via an UndoNxtLSN pointer, creating a breadcrumb trail of rollback progress.

Why this matters: if the system crashes *during* rollback, restart can follow the UndoNxtLSN chain to skip already-undone work. Without this, you get the nightmare scenario where the same update gets undone, redone, and undone again across repeated failures — a problem that actually happened in production IMS and DB2 systems.

> "By appropriate chaining of the log records written during rollbacks to those written during forward progress, a bounded amount of logging is ensured during rollbacks even in the face of repeated failures during restart or of nested rollbacks."

> "In IMS, [the system] may undo the same non-CLR multiple times... in AS/400, DB2 and NonStop SQL, [the system] may also undo CLRs one or more times — these have caused severe problems in real-life customer situations."

This is one of those passages where reading between the lines, you can hear the sound of pagers going off at 3 AM. The authors are polite, but they're describing production disasters that ARIES prevents.

### 3. Three-Pass Restart

After a crash, ARIES runs three passes over the log:

- **Analysis**: Scan from the last checkpoint forward. Rebuild the dirty page table (which pages might have unflushed updates) and the transaction table (which transactions were active, committed, or aborted). Determine the RedoLSN — the earliest LSN that might need redo.

- **Redo**: Starting from RedoLSN, reapply every update whose effects might not be on disk. Per-page LSN comparison makes this idempotent. Pages not in the buffer pool are fetched on demand.

- **Undo**: Starting from the end of the log and working backward, roll back all loser transactions. For each undo, write a CLR. The UndoNxtLSN pointer skips already-undone work if the undo is interrupted.

### 4. Physiological Logging

ARIES uses **page-oriented redo** (you can redo an update by looking at a single page — fast, parallelizable) but **logical undo** (the undo action doesn't have to be the exact physical inverse of the original operation). This is brilliant: redo happens during crash recovery when you want raw speed; undo happens during normal processing and needs to preserve concurrency. Different constraints, different strategies.

> "Logical undo... allows very high concurrency to be supported."

## What ARIES Made Possible

The paper didn't just describe recovery — it described recovery that could coexist with features real database products needed:

- **Fine-granularity locking** (record-level, not page-level)
- **Partial rollbacks** (savepoints within transactions)
- **Operation logging** (increment/decrement without exclusive locks — two transactions can increment the same counter concurrently)
- **Fuzzy checkpoints** (no need to quiesce the system)
- **Media recovery** (restore from fuzzy image copies + replay log)
- **Selective restart** (bring online objects back immediately, defer offline objects)
- **Objects of varying length** (not constrained to page boundaries)
- **Parallelism during restart** (redo is page-oriented and can be parallelized)

## The Comparison Section Is a Masterclass

Section 11 of the paper compares ARIES to every major WAL-based system of the era. It's worth reading for the methodology alone: clear criteria, precise descriptions of where each system fails, and honest acknowledgment of ARIES's own limitations. The comparison with System R's shadow paging is particularly sharp — shadow paging was elegant for its time but fundamentally incompatible with WAL, fine-granularity locking, and variable-length objects.

The authors note that WAL had been proposed earlier (by Gray et al.) but that existing WAL implementations all had significant flaws. ARIES was the first to get all the details right simultaneously.

## Why This Paper Matters Now

Every time you `COMMIT` in PostgreSQL and trust that your data survives a power failure, you're relying on ARIES-derived machinery. The paper is also a masterclass in **systems design communication**: 69 pages, dense but clear, pseudocode throughout, explicit comparisons with alternatives, and honest about edge cases and limitations.

For the agentic development world: ARIES is a reminder that the hardest problems in computing aren't about intelligence — they're about **maintaining invariants across unpredictable failure modes**. Every agent orchestration system that claims durability or reliability is reinventing a tiny subset of what ARIES solved in 1992, usually badly. If you're building an agent with persistent state, read this paper before you invent your own recovery scheme.

ARIES is the origin point of an idea that grew far beyond crash recovery. Jay Kreps traces this arc in [[The Log — Unifying Abstraction for Real-Time Data]]: the WAL evolved from an ACID implementation detail into the mechanism for database replication (Oracle, MySQL, PostgreSQL all ship logs to replicas), then into the general-purpose data integration primitive that powers Kafka and the modern streaming infrastructure. The log that ARIES made reliable enough to trust with crash recovery turned out to be reliable enough to trust with everything else.

The companion to ARIES is [[How to Corrupt an SQLite Database]] — where ARIES defines how recovery *works*, that page catalogs every way it *fails*. Together they bracket the trust boundary: the algorithm that guarantees consistency, and the environmental failures (lying hardware, broken filesystems, application bugs) that breach the guarantee.

The paper also demonstrates that **you ship the details, not just the idea**. WAL and logging existed before ARIES. What ARIES contributed was getting every interaction right: LSNs on pages, CLR chaining, the three-pass structure, the separation of page-oriented redo from logical undo. The whole is greater than the sum, but only because the sum was computed correctly.

## Key Themes

#paper #database #recovery #transaction #WAL #systems-design #fundamentals

---

*Sources: [[summary/aries]]*
*Last updated: 2026-07-05*
*Original: ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992, pp. 94–162. Authors: C. Mohan, Don Haderle, Bruce Lindsay, Hamid Pirahesh, Peter Schwarz (IBM Almaden Research Center and IBM Santa Teresa Laboratory).*
