# Zero-Downtime PostgreSQL Migrations — A Production Playbook

BoundLayer's engineering guide treats zero-downtime schema change not as a Postgres feature but as a release discipline: expand-contract as the spine, lock-queue awareness as the real hazard, batched idempotent backfills as production workloads, and forward repair over blind rollback — wrapped in an ownership model (author, compatibility owner, abort authority, cleanup owner) that turns migration into continuous delivery rather than a maintenance-window event.

---

## What it argues

A migration that is instant on a laptop can take down production, and the SQL is rarely the cause — the danger lives in the surroundings: long transactions, lock queues, connection pools, replicas, and multiple application versions mid-rolling-deploy. The article's best definition is negative: "The database stayed online" is not success. Success is service-level — error rate flat, p95/p99 within limits, replication lag bounded, both app versions valid, and a way to pause or reverse without improvisation.

## Key quotes

> "A schema change that has not even started may cause an expanding production outage."

The lock queue, not the DDL, is the killer: a migration waiting on a long-running transaction makes every new application query queue behind *it*. This justifies the article's most practical habit — a short `lock_timeout` plus bounded retries, so "could not safely acquire the lock" stays distinct from "valid work needs more time".

> "The contract phase must have an owner and a scheduled date. Otherwise temporary columns, dual writes, and unused indexes accumulate into permanent complexity."

Expand-contract is well-known, but the failure mode named here is the honest one: the contract phase is invisible work with no champion, so compat code fossilises. Assigning cleanup ownership *before* the expand begins is the operational fix.

> "A syntactically successful cast may still be semantically wrong."

On type changes: currency, time zones, enums and precision are semantic minefields where Postgres happily converts garbage. The prescribed discipline — quarantine unconvertible values, compare representations, reconcile business invariants — treats migration as data QA, not DDL.

> "Do not run one unbounded update across a large table."

Backfill craft: committed batches by stable key with `FOR UPDATE SKIP LOCKED`, checkpoints, throttling, auto-pause on degraded metrics, and an explicit ban on offset pagination (concurrent changes and large offsets make progress unreliable — see [[SQL Pagination — Offset vs Seek Method]] for why offsets break in general).

## Key themes

#concept #pattern #tool

- **Expand-contract as release sequencing** — five phases, each separately deployable and observable; the rename-in-place and one-shot ALTER are the anti-patterns.
- **Lock-time economics** — the cost of DDL is the lock queue, so bound acquisition, split `NOT VALID` from `VALIDATE` into separate transactions, and treat `CREATE INDEX CONCURRENTLY`'s invalid-index residue as a known cleanup duty.
- **Evidence over row counts** — "declaring success when row count reaches zero" is a named failure mode; reconcile business invariants and instrument old/new divergence before contracting.
- **Ownership and abort conditions** — a migration is unfinished engineering until someone can name who may stop it and on which thresholds.

## Opinionated take

This is a vendor post that earns its length. Nothing here is new to a database veteran — expand-contract dates back decades — but the article's value is packaging the whole discipline as a checklist that survives contact with a real change: preflight questions, per-phase dashboards, a CI compatibility matrix (old app against new schema, mixed versions under dual-write), and a documented point of no easy return. The weakest section is rollback: after destructive contraction, the honest answer is "you can't", and the guidance ("delay destructive cleanup until rollback is no longer operationally necessary") is really just "don't do dangerous things until you're sure" dressed as a runbook. Also missing: any quantified numbers for batch sizes or timeouts beyond illustrative ones, and any acknowledgement that NOT VALID/VALIDATE semantics vary across Postgres versions — though it does hedge on exactly that point. The final takeaway is the right one: "The individual SQL commands matter, but the release sequence matters more."

## Relations

This source strengthens [[DocDB — Stripe's Zero-Downtime Database]], which describes Stripe's live system for exactly this problem — here is the portable playbook version of what Stripe built bespoke. It nuances [[Migrations — The Sole Scalable Fix to Tech Debt]] by supplying the mechanics for the *safe* part of migration practice (expand-contract, bounded backfills) that essay takes as given. It complicates [[Postgres Transactions Are a Distributed Systems Superpower]]: the same transactional machinery that makes Postgres powerful is what makes dual-write divergence and long-transaction lock queues dangerous during schema change. And it pairs naturally with [[Why Are Databases So Hard]], which frames the correctness/availability tension this playbook is designed to manage operationally.

---
*Sources: [[raw/zero-downtime-postgresql-migrations]], [[summary/zero-downtime-postgresql-migrations]]*
*Last updated: 2026-10-18*
