---
url: https://www.dbos.dev/blog/co-locating-workflow-state-with-your-data
title: "Postgres Transactions are a Distributed Systems Superpower"
author: Peter Kraft & Qian Li
date_fetched: 2026-07-05
date_published: 2026-06-15
site: DBOS Blog
category: DBOS Architecture
---

# Postgres Transactions are a Distributed Systems Superpower

**Authors:** Peter Kraft & Qian Li
**Publication Date:** June 15, 2026
**Category:** DBOS Architecture

## Core Thesis

Co-locating workflow engine state with application data inside the same Postgres database — rather than keeping them in separate systems — enables atomic transactions across workflow checkpoints and business data. The authors clarify a prior post: they didn't just mean "use a workflow engine that stores state in Postgres" — they meant "your workflow system can, and often should, live inside the same Postgres database as your application."

"In distributed systems, co-location is a superpower." When both live in the same database, they "can be updated in the same database transaction," eliminating partial failures.

## Key Argument #1: Idempotency with Transactional Steps

Durable workflows checkpoint step results after completion. But a workflow can be interrupted after finishing a step but before recording the checkpoint. On recovery, it re-executes that step — meaning "durable workflows alone do not solve the idempotency problem." Example: crediting $100, crashing, then re-executing would credit $200.

Traditional solution: application-level bookkeeping (e.g., an `applied_payments` table). This is cumbersome.

Co-located approach: the workflow engine writes the checkpoint *and* performs the database update in the same database transaction. The step executes using a transaction provided by the engine — the step's DB updates and the checkpoint commit atomically.

Result: exactly-once semantics. If the transaction commits, both the update and checkpoint are durable. If any failure occurs before commit, everything rolls back. "Transactional steps no longer need application-level idempotency logic or bookkeeping tables."

## Key Argument #2: Atomicity with a Transactional Workflow Outbox

Problem: reliably updating a database *and* sending a notification (e.g., triggering a warehouse fulfillment workflow when an order is placed). These operations need atomicity — "they either both happen or neither do, even if there are failures."

Traditional solution: the transactional outbox pattern — a separate "outbox" table stores messages; a background process polls and delivers them. This guarantees atomicity via a single transaction but "introduces additional operational complexity" — polling infrastructure, retries, monitoring, and reconciliation jobs.

Co-located approach: use a Postgres user-defined function (UDF) called `enqueue_workflow` within the same transaction as the application update. The workflow is a database row containing name, queue, and input. The UDF creates this row atomically with the user's update. A worker then dequeues and executes asynchronously. Same principles as traditional outbox but eliminates manual outbox management and polling infrastructure.

## Related Articles

- "What's New in DBOS - June 2026" (Qian Li, Jun 18, 2026)
- "Just Use Postgres for Task Queues" (Qian Li & Peter Kraft, Jun 2, 2026)
