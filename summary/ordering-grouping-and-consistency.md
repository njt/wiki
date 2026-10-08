---
url: https://event-driven.io/en/ordering-grouping-and-consistency/
title: "Ordering, Grouping and Consistency in Messaging Systems"
author: Oskar Dudycz
date_fetched: 2026-10-08
date_published: unknown
topics:
  - distributed-systems
  - software-engineering-craft
---

Oskar Dudycz continues his Queue Broker series by connecting message grouping to idempotency. He first surveys how major messaging systems handle ordering within groups: Amazon SQS FIFO (message group IDs, at a throughput cost), Azure Service Bus (sessions with locks), Kafka (partitioning, with rebalancing disruption), and RabbitMQ (ordering only within a single queue; grouping needs one queue per group or consistent-hashing routing, both of which scale badly).

The core move: rather than relying on a distributed lock (Redis, relational DB) for idempotency, build a thread-safe in-memory idempotency key store on top of his Queue Broker's single-writer scheduling. The broker gains a `taskGroupId` option; `processQueue` skips tasks whose group is currently active, leaving them in place in the queue. Serialized per-group processing means the check-then-lock on an idempotency key is race-free without any external lock manager.

He walks through an example scheduling trace, gives the full TypeScript implementation, and is honest about trade-offs: O(n) linear scan for eligible tasks, head-of-line blocking within hot groups, asynchronous cleanup lag, and fairness problems for ungrouped tasks. His verdict: don't ship it to production — it's a learning exercise — but the same trade-offs transfer to real queues. He also reiterates his preference for handling idempotency in business logic rather than generic infrastructure, since business rules change and generic handling taxes every call.
