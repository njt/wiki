# Materialized Views Are Obviously Useful

Sophie Alpert argues that incremental view maintenance -- databases automatically keeping derived data in sync with base tables -- is so obviously useful that it will be standard within a decade. The argument is made through escalating pain: a simple `count(*)` query becomes a Redis cache, then a Lua-scripted increment/decrement system, then a Kafka pipeline, each layer adding failure modes without adding correctness.

---

## Key Quotes

> "Correctness of my system today depends not only on the code being correct right now but also on my code having done the correct thing at every point in the past."

> "A decade from now, most database systems will have a version of this built in."

## Key Themes

#databases #caching #correctness #declarative #simplicity

## The Argument

Alpert walks through a familiar trajectory that most backend engineers have lived:

1. **Start simple.** `SELECT count(1) FROM tasks WHERE project_id = $1` works until it doesn't scale.
2. **Add a cache.** Redis with TTL. Fast, but now users see stale counts after creating tasks. The cache and the database disagree.
3. **Make the cache smart.** Increment on insert, decrement on delete. Requires Lua scripts for atomicity. Breaks when tasks move between projects (must update two counters). Breaks permanently when a server crashes between the database write and the cache update.
4. **Add infrastructure.** Kafka for reliable event delivery, or wrap everything in database transactions. Each fix introduces new failure modes and new infrastructure to operate.
5. **Declare the answer.** `CREATE MATERIALIZED VIEW projects_task_count AS SELECT project_id, count(1) FROM tasks GROUP BY project_id` -- and let the database maintain it automatically via dataflow graph analysis.

The escalation is the point. Every step from 2 to 4 is application-level code reimplementing what a database should provide natively. The complexity is accidental, not essential.

## Critical Analysis

Alpert is right that this is obviously useful, and the escalating-pain narrative is effective pedagogy. But the piece understates two things.

**The hard part isn't the vision, it's the implementation.** Postgres has had materialized views since 9.3 (2013). They require manual `REFRESH`. Oracle has `ON COMMIT` materialized views but with severe restrictions on supported query shapes. The reason "most database systems" don't do this well isn't lack of imagination -- it's that incremental view maintenance for arbitrary SQL is genuinely hard. Aggregations with GROUP BY are tractable; joins across large tables with deletions and updates are not. The gap between the declarative dream and production reality is where a decade of database engineering lives.

**The application-level cache isn't always wrong.** Sometimes you want a cache that's intentionally approximate (showing slightly stale counts is fine; spending zero query budget on counts is the goal). Sometimes the derived data crosses service boundaries where no single database owns both sides. The article frames all application-level synchronization as accidental complexity, but some of it is architectural choice in a distributed system.

That said, the core thesis holds: developers spend enormous effort hand-rolling synchronization that databases should handle declaratively. The fact that this is hard to implement doesn't make it less obviously useful -- it makes it a high-value problem worth solving.

This connects to [[Correct by Construction]] -- both argue that correctness should be structural (declared in the schema/view definition) rather than procedural (maintained by application code that must be correct at every point in time). It also connects to [[Designing a Passively Safe API]] -- Alpert's escalating failure modes (crash between DB write and cache update) are exactly the kind of partial-failure states that passive safety patterns address. And it echoes [[Simplicity in the Age of AI-Assisted]] -- the Redis-to-Kafka escalation is a textbook case of inherited complexity that exists because the right abstraction wasn't available.

The piece is also relevant to [[Databases and Data]] -- incremental view maintenance is exactly the kind of "streaming and real-time" capability that synthesis page identifies as missing from the wiki's database coverage.

## Cross-Links

- [[Correct by Construction]] -- structural correctness vs. procedural maintenance
- [[Designing a Passively Safe API]] -- the partial-failure problem Alpert describes
- [[Simplicity in the Age of AI-Assisted]] -- inherited complexity from missing abstractions
- [[Databases and Data]] -- fills the "streaming and real-time" gap
- [[Write Snapshot Isolation]] -- another case where correctness should be built in, not bolted on
- [[Dapper Performance Trap]] -- database subtleties that application code gets wrong

---
*Sources: [[summary/materialized-views-are-obviously-useful]]*
*Last updated: 2026-05-14*
