---
url: https://shopify.engineering/scaling-inventory-reservations
title: "We replaced Redis with MySQL for inventory reservations—and it scaled"
author: Emilie Noel
date_fetched: 2026-05-31
date_published: 2026-05-12
---

Shopify replaced Redis with MySQL for inventory reservations and survived Black Friday 2025 at $5.1M/minute peak sales. The article walks through the technical decisions: one-row-per-unit design with SKIP LOCKED, bounded inventory pools capped at 1,000 rows per item/location, composite primary keys to reduce InnoDB row locks from two to one, READ COMMITTED isolation to avoid gap/supremum locks blocking replenishment, consistent lock ordering to prevent deadlocks, and UNION ALL batching for multi-item carts.

The real bottleneck turned out to be connection pool pressure from other checkout processes, not reservation throughput itself. Per-caller SQL comment tagging (`/* conn_tag:checkout_completion */`) via ProxySQL revealed which processes were holding connections too long — they hadn't been optimized because they weren't the first to hit the limit. After cleanup: 50% of reads and 33% of transactions removed from the primary database. Writer CPU stayed under 50% and reader CPU under 16% during flash sales.

Dual-write shadow mode (Redis + MySQL, Redis as source of truth) enabled a gradual, pod-by-pod, reversible cutover with a kill switch for instant reversion. Ran on production traffic until correctness and performance were validated.

Key technical decisions:
- One row per sellable unit (not quantity columns) — enables SKIP LOCKED concurrency
- Capped pool of 1,000 rows per item/location to prevent table bloat; inline replenishment on exhaustion
- Composite PK (shop_id, inventory_item_id, inventory_group_id, id) — filter columns in the PK, one lock per row instead of two
- READ COMMITTED isolation — first non-default isolation level in this codebase; eliminates gap locks on empty tables
- Consistent lock ordering: reserve DELETEs from reservation_units first, then INSERTs into reserved_quantities
- UNION ALL batching for multi-item cart reservations — one round trip instead of N
- Per-caller SQL comment tagging via ProxySQL for connection hold-time diagnostics
- Dual-write shadow mode with Redis as source of truth during validation; kill switch for instant reversion

Key quotes:
- "The hardest lesson wasn't about database design. It was discovering that the real bottleneck wasn't what we were observing and measuring."
- "Reservations were the straw that broke the camel's back — not because reservations were slow, but because the pool was already near depletion."
- "If you're reaching for Redis, Kafka, or a custom coordination layer for high-throughput mutual exclusion, your existing database might already be enough."
- "This wasn't about making reservations fast. It was about making them safe neighbors."
- "If the numbers don't add up — low CPU but high queuing — instrument the full path. The answer is often in the plumbing, not the engine."

Related references:
- Jahfer Husain's guide to InnoDB locking (cited in article)
- 37signals' approach to database-backed load distribution (cited as inspiration)
- MySQL 8 SKIP LOCKED documentation

Scale context: Shopify powers over 14% of U.S. ecommerce. Black Friday 2025: $5.1M/minute peak, 11% YoY increase.
