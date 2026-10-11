---
url: https://btmc.substack.com/p/transaction-oriented-programming
title: "Transaction-Oriented Programming"
author: btmc (Barry Meijers)
date_fetched: 2026-10-11
date_published: 2026-10-05
topics:
  - software-engineering-craft
  - databases-and-data
---

btmc proposes "Transaction-Oriented Programming": identify the **user-relevant units of work** in an application and implement each as a transaction with ACID properties. The unit is context-dependent — `cd` is one indivisible transaction (which is why `abort()` is a perfectly good error strategy there), `cp` of a directory makes each file a unit (log and continue), while a partial file copy is never a unit except in BitTorrent-style downloads.

The unifying claim about error handling: whatever mechanism you use (exceptions, error codes, result types, error logs), every strategy ends up doing the same job — abort the current transaction and inform the user. So the choice of mechanism matters far less than having a plan for how each unit is made atomic, consistent, isolated, and (where relevant) durable.

He then reads familiar architectures through this lens. Distributed systems get ACID "for free" by leaning on a database; OOP is "local and synchronous distributed software" with no database to fall back on, so a web of locally-consistent objects is not consistent as a whole (Smalltalk survives only via image durability). Functional Core, Imperative Shell works because the core is trivially a transaction — app state is never touched on error — and the same outcome could come from double-buffered state or undo. Immediate-mode/reactive UIs reduce inconsistency by deriving state, but derivation is just the easiest mechanism, not the essence.

He sketches a Python prototype (composable `Transaction` base class with `_task`/`_undo`, `Sequence` and `Optional` combinators) and a hypothetical transaction-oriented language with `transaction` blocks, `on execute`/`on abort`/`on commit` clauses for non-undoable effects, and `?`/`!` operators for partial success and commit-through-failure. Conclusion: we may not need a new paradigm, but evaluating architectures by how easily they let you implement transactions — and rephrasing error-handling correctness in transaction terms — is the useful discipline. Ted Codd gets the final bow; OOP takes the blame.
