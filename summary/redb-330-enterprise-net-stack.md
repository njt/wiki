---
url: https://dev.to/rinat_kozin/redb-330-an-enterprise-net-stack-you-actually-own-typed-store-a-homegrown-apache-camel-and-3gd1
title: "redb 3.3.0: an enterprise .NET stack you actually own — typed store, a homegrown Apache Camel, and a runtime with a dashboard (all free)"
author: Rinat Kozin
date_fetched: 2026-07-18
date_published: 2026-07-09
topics:
  - databases-and-data
---

Rinat Kozin announces the 3.3.0 release of redb, a three-layer open-source .NET stack: **redb** (typed POCO store with LINQ, no migrations, multiple database providers), **redb.Route** (integration engine in the Apache Camel mold with route DSL and 30+ connectors), and **redb.Tsak** (runtime with dashboard, hot-reload, and clustering).

Two new transports land: Amazon SQS/SNS (`sqs://`, `sns://`), built on the native AWS SDK v4 with long-polling, competing consumers, FIFO support, and transactional acknowledgements; and Telegram (`telegram://`), a long-polling consumer with fluent DSL for sending messages, documents, photos, and inline keyboards.

A complete end-to-end RAG pipeline is introduced: `knowledge://` ingests and chunks documents, `embed://` produces vectors via registered embedding providers, and `.Knowledge(collection, k)` retrieves top-K chunks to inject into an LLM prompt. A ready-made `knowledge_search` tool lets models decide when to search. Semantic search uses cosine similarity when an embedding provider is present, falling back to keyword `LIKE` matching otherwise. A JSON-escaping fix (switching to `UnsafeRelaxedJsonEscaping`) addresses a ~6× token blow-up for non-ASCII text.

The `.Threads(N)` DSL stage caps a route section at N parallel workers without needing a named SEDA endpoint. A significant fix makes `ConcurrentConsumers(N)` actually parallelize — it was silently serial across RabbitMQ, Kafka, AMQP 1.0, and IBM MQ due to dispatch-concurrency defaults and blocking receive loops. Per-exchange DB connections replace a captive singleton `IRedbService`, resolving "command already in progress" errors under concurrency.

Several bug fixes ship: a connection-pool leak under transaction-dispose failures, a query-parser fix for `array.Contains(x)` on .NET 9, and fail-fast detection of multi-threaded use of a single `IRedbService`. The Pro edition — change tracking, bulk operations, analytics, and cluster features — is now free with "no catch": no license keys, sign-up, or node limits.

Kozin positions 3.3.0 as the maturity release: "the release after which you can actually push redb.Route hard and stop chasing flaky connection errors."

---
*Sources: [[raw/redb-330-enterprise-net-stack]]*
*Last updated: 2026-08-01*
