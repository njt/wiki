---
title: "Write Snapshot Isolation"
url: https://remy.wang/blog/si.html
date_fetched: 2026-05-14
section: "Databases and Data"
---

# Simple and Correct Snapshot Isolation

## Overview
This article examines snapshot isolation (SI) in database systems, highlighting its limitations and proposing write-snapshot isolation (WSI) as a superior alternative that guarantees serializability.

## Key Arguments

**The Problem with Snapshot Isolation:**
SI avoids write-write conflicts by aborting transactions when they attempt to overwrite recently modified data. However, this approach produces two problematic outcomes: it rejects some serializable executions (false negatives) and permits non-serializable ones (false positives).

The author illustrates this paradox through two examples. In the first scenario, SI aborts a valid serial execution. In the second, SI permits an execution that no serial ordering could produce, since values are incremented by different amounts depending on transaction sequence.

**The Root Cause:**
SI focuses on preventing stale writes, but this misidentifies the actual problem. The relevant concern is stale reads—when a transaction reads outdated values and bases writes on that information, database inconsistencies follow.

**The WSI Solution:**
WSI replaces write-write conflict detection with read-staleness detection. Rather than aborting when transactions write to recently modified items, WSI aborts when transactions commit after reading values that have since been updated.

## Notable Quotes

"SI both forbids certain correct executions (false negatives) and permits incorrect ones (false positives)"

"What we really should be worrying about is stale reads"

"WSI guarantees serializability...by changing one single line of code"

## Main Conclusions

- WSI achieves strict serializability through minimal implementation changes
- Adoption remains limited due to historical timing (PostgreSQL already adopted SSI) and implementation complexity
- WSI merits consideration for new database systems, though it necessarily aborts more transactions than standard SI
- The elegance of WSI contrasts with SSI's "bolt-on" approach to fixing SI's fundamental flaws
