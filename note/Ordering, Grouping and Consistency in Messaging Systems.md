# Ordering, Grouping and Consistency in Messaging Systems

Oskar Dudycz merges two threads from his Queue Broker series — grouped single-writer scheduling and idempotency handling — into a working, thread-safe in-memory idempotency key store, pausing along the way to survey how SQS FIFO, Azure Service Bus, Kafka and RabbitMQ each solve (or fail to solve) message grouping.

---

## What it argues

**Grouping is the quiet workhorse of async systems.** Bank withdrawals before deposits, two check-in commands for one guest, an IoT door event racing a door command — the failures are all the same shape: related operations on shared state that run concurrently. Grouping serializes operations within a group (account, order, device) while letting different groups run in parallel.

**Every broker implements this differently, and none for free:**

- SQS FIFO: message group IDs, but throughput drops and a slow task head-of-line blocks its whole group.
- Azure Service Bus: sessions with locks — robust, but lock management adds complexity.
- Kafka: partitioning, i.e. a *physical* split; poor partition keys skew load, and rebalancing disrupts processing.
- RabbitMQ: ordering only within a single queue; round-robin consumers destroy it. Grouping means one queue per group (thousands of queues for a `userId` key) or consistent-hashing routing.

**The original contribution** is the merge: instead of a distributed lock (Redis) for idempotency, he uses his Queue Broker's single-writer scheduling. Tasks get an optional `taskGroupId`; the processing loop only dequeues tasks whose group isn't active, leaving blocked tasks in place. Because scheduling is single-writer, the check-then-add on an idempotency key is atomic by construction — no lock manager needed. The store is a three-verb interface: `tryLock`, `accept`, `release`.

## Key quotes

> "Grouping allows related operations to be processed sequentially while operations on different groups run concurrently."

The entire pattern in one sentence — partition concurrency by the entity you actually care about, not by worker count.

> "I recommend handling idempotency using business logic, as it's highly dependent on the business rules. And those rules tend to change. If we use generic handling, we're always adding additional overhead, even if most of our applications rarely have idempotency issues."

A deliberately unfashionable stance: against generic middleware, on the grounds that generic machinery taxes the 95% of calls that never race. Consistent with his larger theme that infrastructure should serve business semantics, not replace them.

> "Please, please don't take it to production."

He names his own trade-offs — O(n) scan for eligible tasks, hot-group head-of-line blocking, async cleanup lag, ungrouped-task unfairness — and notes the fix for the scan (a queue per key) walks straight back into the RabbitMQ scaling trap he'd just described. That self-awareness is the article's best feature.

## Themes

#concept #pattern #distributed-systems

## Opinionated take

The genuinely interesting idea is that **single-writer scheduling is a lock you already own**. Most idempotency advice immediately reaches for Redis or a database row lock; Dudycz shows that if your scheduler guarantees one-at-a-time processing per group, the race window simply never opens in-process. That's elegant — but the "don't ship this" caveat is doing real work. A Map of locked keys dies with the process, so this only works where the store's lifetime matches the work's lifetime, and the article never states that boundary explicitly. It's a great teaching construction that readers may misread as production advice.

The broker survey is also refreshingly concrete: RabbitMQ's lack of native grouping and the resulting queue-per-group scaling cliff is the kind of operational detail usually glossed over in comparison tables. The recurring lesson across all four brokers — ordering guarantees cost throughput somewhere, and the cost lands on your least-noticeable axis (latency in a group, uneven partitions, queue explosion) — is the part worth remembering even if the TypeScript never gets deployed.

## Related pages

- [[Idempotency Is Easy Until the Second Request Is Different]] — this source's companion problem from the other direction: Dochia's article treats concurrent retries and key semantics in depth; Dudycz's grouped queue is one concrete mechanism that closes exactly the race window Dochia identifies, but his key store remembers less than her "useful version" demands.
- [[State-Oriented Consistency]] — the Keel IoT team's per-piece-state consistency question is the design reflex behind this article's grouping decision: pick guarantees per group key (account, order, device) rather than one policy for the whole system.
- [[Queues Don't Fix Overload]] — a useful corrective pairing: where that piece warns queues aren't a load-fixing tool, this one shows what queues *are* for — ordering, grouping, and exclusive processing guarantees.
- [[The Log — Unifying Abstraction for Real-Time Data]] — Kafka's partitioning model here is the operational face of the log abstraction; the rebalancing and skew costs described are what that abstraction buys its guarantees with.

---
*Sources: [[raw/ordering-grouping-and-consistency]], [[summary/ordering-grouping-and-consistency]]*
*Last updated: 2026-10-08*
