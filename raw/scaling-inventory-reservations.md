---
url: https://shopify.engineering/scaling-inventory-reservations
title: "We replaced Redis with MySQL for inventory reservations—and it scaled"
author: Emilie Noel
date_fetched: 2026-05-31
date_published: 2026-05-12
---

Shopify replaced Redis with MySQL for inventory reservations using MySQL's SKIP LOCKED feature. The article walks through the technical decisions: one-row-per-unit design, bounded inventory pools, composite primary keys to reduce row locks, READ COMMITTED isolation to avoid gap locks, consistent lock ordering to prevent deadlocks, and UNION ALL batching for multi-item carts. The real bottleneck turned out to be connection pool pressure from other checkout processes, not reservation throughput itself. Per-caller SQL tagging via ProxySQL revealed which processes were holding connections too long. After cleanup, writer CPU stayed under 50% and reader CPU under 16% during flash sales. Dual-write shadow mode enabled a gradual, reversible cutover from Redis to MySQL.

Key technical decisions:
- One row per sellable unit (not quantity columns) — enables SKIP LOCKED
- Capped pool of 1,000 rows per item/location to prevent table bloat
- Inline replenishment when the pool empties during flash sales
- Composite PK (shop_id, inventory_item_id, inventory_group_id, id) to avoid InnoDB locking secondary index + clustered index
- READ COMMITTED isolation level to prevent gap locks blocking replenishment inserts
- Consistent lock ordering: reserve DELETEs from units first, then INSERTs into reserved_quantities
- UNION ALL batching for multi-item cart reservations
- Per-caller SQL comment tagging via ProxySQL for connection diagnostics
- Dual-write shadow mode with Redis as source of truth during validation
- Kill switch for instant reversion
