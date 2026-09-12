# Write Snapshot Isolation

Standard snapshot isolation (SI) has a fundamental misdiagnosis at its core: it checks for stale writes when it should be checking for stale reads. Write Snapshot Isolation (WSI) fixes this with a single conceptual change, achieving serializability where SI cannot. The elegance is striking -- one line of code separates "sometimes wrong" from "always correct."

---

## Key Quotes

> "SI both forbids certain correct executions (false negatives) and permits incorrect ones (false positives)"

> "What we really should be worrying about is stale reads"

> "WSI guarantees serializability...by changing one single line of code"

## Key Themes

#database #data-quality

The core insight -- that SI's write-write conflict detection is aimed at the wrong target -- is a beautiful example of how a small conceptual reframe can fix a fundamental design flaw. SI prevents two transactions from writing to the same item, but the real danger is when a transaction reads stale data and then makes a write decision based on that outdated information.

WSI flips the check: instead of aborting when two transactions write to the same item, it aborts when a transaction tries to commit after reading a value that has since been updated by another committed transaction. This is read-staleness detection rather than write-conflict detection.

The connection to [[Correct by Construction]] is worth noting: both pieces argue that correctness should be structural, not bolted on. WSI builds serializability into the isolation mechanism itself, rather than adding SSI-style detection as an afterthought.

## Critical Analysis

This is a clean, well-argued technical piece. The two examples that show SI both rejecting valid executions and permitting invalid ones are compelling. The practical question -- why hasn't WSI been adopted more widely? -- gets an honest answer: PostgreSQL shipped SSI first, and momentum matters more than elegance in database adoption. WSI necessarily aborts more transactions than plain SI (because it's actually catching real problems), which makes it a harder sell to teams that prioritize throughput over correctness. The tradeoff is real, but for systems where serializability matters, WSI is the cleaner path.

---
*Sources: [[summary/write-snapshot-isolation]]*
*Last updated: 2026-05-14*
