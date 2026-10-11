# Transaction-Oriented Programming

btmc argues that reliable software is built out of **user-relevant units of work** implemented as ACID transactions — and that this one idea explains why `abort()` is fine for batch tools but not interactive apps, why Functional Core/Imperative Shell works, why OOP struggles with consistency, and what a hypothetical "transaction-oriented language" might look like. Self-admittedly untested in production, but grounded in ideas that already work.

---

## The Core Idea

A **user-relevant unit of work** is deliberately fuzzy, defined by example: the whole of `cd` (you changed directories or you didn't), but a single file within `cp -r` of a directory (one bad file shouldn't kill the whole copy), and *not* a partial file write — unless you're BitTorrent, where partial chunks are the point. The unit is contextual; the invariant is not: error handling happens around the unit, and each unit is a transaction.

## Key Quotes

> "Whichever error handling mechanism (exceptions, error codes, etc.) you use, they will all ultimately have to accomplish the same thing: abort the current transaction and inform the user."

This defuses a perennial religious war in one move. Monadic error codes help you *remember* to check, which is real but incidental; the destination is identical. Pick whichever mechanism is simplest for the task at hand.

> "Object-oriented software ... is just a local and synchronous form of distributed software. The same problem is there, but now there's no database to fallback to, so it is a disaster in this regard."

The sharpest line in the piece. A web of objects each maintaining its own invariants is not consistent as a whole — exactly the "local consistency ≠ global consistency" trap that distributed systems people learned the hard way. Smalltalk only survives it because the image gives you durability-time-travel.

> "If you are doing 'Functional Core, Imperative Shell', you're already reaping most of the benefits an imaginary 'Transaction-Oriented Programming' paradigm would give you."

And, importantly, the functional part "matters very little" — what matters is that app state is never touched on the error path. Double-buffered state or undo-log would do the same job. Derived UI state (immediate mode, reactive) is likewise "just the easiest way to achieve the intended outcome."

> "Ted Codd was a genius and we've all suffered massively for not listening to him and building software using the relational model, instead chasing silly ideas like object-oriented programming and garbage like SQL."

The essay's shot of vinegar. Tongue-in-cheek on SQL, less so on OOP — and the body of the argument (communicating objects with isolated state as degenerate distributed systems) is a real critique, not just a jab.

## The Prototype and the Language Sketch

The Python prototype is the essay's weak link taken seriously: a `Transaction` ABC with `_task`/`_undo`, composed via `Sequence` (undo completed children on failure) and `Optional` (swallow errors, undo only on parent failure). The code is buggy (`return true`, `self.undo()` vs `_undo()`), which is fine for a sketch but underlines that the interesting content is the conceptual bucket, not the implementation. The language sketch is more evocative: `on execute` / `on abort` / `on commit` clauses so that non-undoable effects (printing) queue until the parent commits, plus `?` (optional) and `!` (commit-through) operators. He honestly notes the compilation cost: undo stacks and commit queues want arena allocation, à la Go's `defer`.

## Themes

#concept #pattern #software-engineering-craft #databases

## Opinionated Take

This is a genuinely clarifying essay wearing a rough shirt. The insight that error-handling mechanisms are interchangeable *given a transaction boundary* is correct and under-taught; most error-handling debates assume the boundary is whatever the nearest try-block happens to be. The claim that architectures should be graded by "how easy they make it to implement transactions" is a usable heuristic — it explains saga patterns, STM, event sourcing, and FCIS with one yardstick.

The limits are honest ones. "User-relevant" is acknowledged as fuzzy, and that fuzziness is where all the difficulty actually lives: in a real app the units change under you, and effects that can't be undone (payments, emails) break the clean execute/undo symmetry in ways his `on commit` clause only gestures at. Distributed transactions beyond a single process get hand-waved to "the database does it" — sagas, 2PC, and idempotency exist precisely because they often can't. And the OOP polemic overreaches: well-encapsulated aggregate designs are arguably *doing* transaction-oriented thinking at a smaller scale. Still, as a lens — especially in an agent era where long task loops desperately need well-defined abort/undo boundaries — it's a strong candidate for "obvious in retrospect."

## Related Pages

- [[State-Oriented Consistency]] — complicates this source usefully: btmc treats consistency as a transaction-level property to enforce wholesale, while that piece argues consistency is a per-piece-of-state decision; the two together suggest transactions whose *granularity* follows the state's actual needs.
- [[SQLite is All You Need for Durable Workflows]] — strengthens the durability half of ACID for agent-era workloads: SQLite as the cheap way to get transactional durability without a database service, exactly the "database does it" fallback btmc describes.
- [[Durable Execution Without History Replay]] — an alternative answer to btmc's undo-vs-collect question: instead of undoing work or deferring commits, checkpoint the live continuation at each durable boundary so recovery cost tracks needed state, not history.
- [[Durable Actors — Open-Source Durable Objects]] — serialized execution of stateful actors is a concurrency story for the same problem btmc's Isolation property raises for task queues and thread pools.

---
*Sources: [[raw/transaction-oriented-programming]], [[summary/transaction-oriented-programming]]*
*Last updated: 2026-10-11*
