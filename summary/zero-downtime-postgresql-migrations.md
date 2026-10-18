---
url: https://boundlayer.io/blog/zero-downtime-postgresql-migrations
title: "Zero-Downtime PostgreSQL Migrations: A Production Playbook for Safe Schema Changes"
author: BoundLayer Engineering Team
date_fetched: 2026-10-18
date_published: unknown
topics:
  - databases-and-data
  - software-engineering-craft
---

A long, practical playbook for evolving PostgreSQL schemas without taking the service down. Its central reframing: zero downtime is not a Postgres feature but a **release discipline** — the schema and application move together through controlled intermediate states, with old and new versions kept compatible throughout.

The core patterns:

- **Expand-contract (parallel change)** is the spine: add compatible schema, backfill, switch traffic, verify, then remove the old contract — with a named owner and deadline for the contract phase, or temporary compatibility code becomes permanent.
- **Locks are the real hazard.** Many `ALTER TABLE` forms take an `ACCESS EXCLUSIVE` lock, and the lock *queue* behind a waiting migration can cause an outage before the migration even starts. Mitigations: short `lock_timeout` with deliberate retries, `NOT VALID` + separate `VALIDATE CONSTRAINT`, `CREATE INDEX CONCURRENTLY` (with invalid-index cleanup on failure).
- **Column renames and type changes** go through shadow columns and multi-release dual-write rather than a single command; each phase is separately deployable and observable.
- **Backfills are production workloads**: small committed batches selected by stable keys (`FOR UPDATE SKIP LOCKED`, never offset pagination), idempotent, checkpointed, throttled, and paused automatically when service metrics degrade.
- **Rollback planning favours forward repair** — reversing DDL doesn't reverse transformed data; document the point of no easy return and require explicit approval to cross it.
- Success is defined by service-level outcomes (error rate, p95/p99, replication lag, dual-version compatibility), not "the database stayed online". The runbook ends with observability dashboards per migration, production-like rehearsal, a CI compatibility matrix (old app vs new schema etc.), and assigned ownership — author, compatibility owner, abort authority, cleanup owner.

A vendor post (BoundLayer sells exactly this service), but the content is genuinely operational and specific — the strongest single-page treatment of the expand-contract discipline for Postgres.
