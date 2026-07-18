# Redb Ecosystem

A three-layer open-source .NET stack (Apache 2.0) that combines a typed LINQ-native database, an Apache Camel-style integration engine with 30+ connectors, and a clustered runtime with dashboard — all free, including the formerly-paid Pro features. Created by Rinat Kozin.

---

The redb ecosystem competes in a space where .NET enterprise stacks are traditionally expensive and fragmented. Its thesis is that a developer should be able to define a POCO, query it with LINQ, route messages through enterprise integration patterns, and deploy to a clustered runtime — all from NuGet packages with no license keys.

## The Three Layers

**redb (typed store):** A database that generates its schema from your C# classes. No migration files, no `Include()` statements, full LINQ translated server-side. Think Entity Framework without the object-relational impedance mismatch — the object *is* the schema. Providers for Postgres, MSSQL, and SQLite.

**redb.Route (integration engine):** An Apache Camel for .NET. Route DSL: `From("kafka://orders").Process(...).To("sqs://fulfillment")`. 30+ connectors including Kafka, RabbitMQ, HTTP, gRPC, S3, LLM, Telegram, and SQS/SNS. Enterprise Integration Patterns (content-based router, splitter, aggregator, scatter-gather) are first-class DSL steps, not library calls.

**redb.Tsak (runtime):** Turns routes into a production service. Dashboard, hot-reloadable modules, metrics, cluster with coordinator and failover. Ships as Docker images (cosign-signed), NuGet packages, and standalone archives.

## The 3.3.0 Release: Honest Concurrency

The headline of this release isn't the new connectors (SQS, Telegram) or the RAG pipeline — it's that **concurrency was silently broken until now**.

> "`ConcurrentConsumers(N)` was silently giving you no parallelism on the brokers."

This is the kind of admission you rarely see in release notes. The option existed, sized a `SemaphoreSlim`, and did nothing. Across RabbitMQ, Kafka, AMQP 1.0, and IBM MQ, the implementation had different flavors of the same bug: the concurrency primitive was there, but the transport layer wasn't wired to use it. RabbitMQ channels defaulted to dispatch concurrency of 1. AMQP consumers awaited each message inline. Kafka's transacted producer threw on deferred sends.

The fix is per-connector and honest about the blast radius:
- **RabbitMQ:** Channels now open with explicit dispatch concurrency matching `ConcurrentConsumers`
- **AMQP 1.0 / IBM MQ:** N genuine competing consumers, each with its own session/connection (IBM MQ needs this because `Backout()` rolls back the whole connection)
- **Kafka:** New `EnableAutoCommit` option, deferred-send fix

The upgrade warning is refreshingly direct: if your route had `ConcurrentConsumers(N > 1)` and "worked," it was serial. After upgrade it will genuinely parallelize, and "per-queue ordering is no longer preserved."

## The DI Scope Bug: Another Honest Fix

The same release fixes a second systemic issue: `IRedbService` (one non-thread-safe DB connection) was captured as a singleton from the root DI container. Two concurrent exchanges would drive one connection, producing "A command is already in progress" errors. The fix — each exchange gets its own DI scope, its own scoped `IRedbService`, its own pooled connection — is the kind of thing that's obvious once you hear it but took shipping to production to surface.

This release also fixes a connection-pool leak (transaction dispose failure skipped returning the connection to the pool) and several clone leaks (ThreadsProcessor, SedaProducer, WireTapProcessor all leaked exchange clones on failure paths).

The author's summary is accurate: "the release after which you can actually push redb.Route hard and stop chasing flaky connection errors."

## Built-In RAG Without External Infrastructure

Three new DSL primitives close the RAG loop entirely within the routing engine:

1. **`knowledge://`** — ingest documents, chunk deterministically, embed, upsert
2. **`embed://`** — turn text into vectors as a pipeline step
3. **`.Knowledge(collection, k)`** — retrieve top-K chunks and inject them into the LLM prompt

A full RAG bot on Telegram is three route definitions. No separate vector database, no orchestration layer, no Python microservice for embeddings. The knowledge store can fall back to SQL `LIKE` when no embedding provider is registered — it degrades gracefully rather than breaking.

A small but telling detail: they switched to `UnsafeRelaxedJsonEscaping` across LLM serializers because standard JSON escaping was 6×'ing token counts for non-ASCII content. That's the level of attention to production cost that distinguishes a serious tool from a proof of concept.

## `.Threads(N)` — Concurrency as a DSL Stage

Rather than requiring named SEDA endpoints for parallelism, `.Threads(N)` is now a composable route stage:

```csharp
From("mqtt://sensors")
    .Threads(8)
        .Process(ex => HeavyWork(ex))
    .EndThreads();
```

It adapts to the exchange pattern: InOnly gets a worker pool with clone-and-handoff; InOut runs inline under a `SemaphoreSlim` gate so the reply survives. This is good DSL design — the abstraction doesn't leak the implementation detail that fire-and-forget and request-reply need different concurrency strategies.

## The Pro-is-Free Model

Pro packages remain closed-source but are distributed free with "no bookkeeping — no licenses, keys, sign-up, license server, or node/volume limits." This includes change tracking, bulk operations, analytical queries (grouping, window functions), and Tsak cluster features (coordinator, distributed lock, leader election, failover).

It's a land-grab strategy: make the stack indispensable, monetize later (or don't). For .NET shops that have been paying for SQL Server licenses and commercial message brokers, it's a compelling pitch — but the sustainability question is real. A single-developer project giving away an enterprise stack for free is either a labor of love or a bet that hasn't been called yet.

## Critical Assessment

**What's genuinely impressive:**

The ecosystem coherence. redb, redb.Route, and redb.Tsak were designed together — the typed store's `IRedbService` integrates directly into route DSL steps, and the runtime understands routes as first-class deployable modules. This isn't three projects bolted together; it's one design.

The honesty about bugs. The `ConcurrentConsumers(N)` admission — "this option existed and did nothing" — is rare in any ecosystem. Most projects would fix it quietly in a patch release. Making it the headline of a minor version shows a level of engineering integrity that builds trust.

The RAG-as-DSL approach. Building RAG as three route primitives rather than a separate service is genuinely elegant. It treats retrieval as just another integration pattern — which, architecturally, it is.

**What warrants skepticism:**

The single-developer risk. The entire ecosystem appears to be one person's work. The code velocity is impressive but the bus factor is 1. For an "enterprise stack you actually own," the bus factor matters more than the license.

The Pro-is-free sustainability. Closed-source but free-with-no-catch is an unstable equilibrium. Either it monetizes (and the terms change) or it doesn't (and maintenance eventually stops). The .NET ecosystem has seen this pattern before — useful tools that go dormant when the author's circumstances change.

The Camel comparison. Apache Camel has 15+ years of production hardening across thousands of organizations. redb.Route is a spiritual port — it captures the DSL style and EIP patterns — but a spiritual port is not the same thing as 15 years of edge cases found and fixed. The 3.3.0 concurrency bugs are exactly the kind of thing that long-tenured projects have already discovered.

**The bottom line:** For a .NET team that wants typed data access without ORM ceremony, integration patterns without YAML configuration files, and a runtime they can `docker pull` — redb is worth evaluating. The 3.3.0 release removes the biggest production blockers. But "enterprise stack you actually own" is doing a lot of work when the stack is maintained by one person. Own the code, yes. Own the risk, too.

---

## Key Themes

- **#pattern** — Enterprise Integration Patterns as a first-class DSL rather than a library. The route is the architecture diagram.
- **#tool** — Typed store with no migrations: schema-from-code as an alternative to code-from-schema. The POCO is the source of truth.
- **#concept** — Concurrency honesty: admitting a feature was silently broken and fixing it loudly is a trust-building pattern more projects should adopt.
- **#concept** — RAG as integration pattern: retrieval-augmented generation doesn't need a separate service; it's a pipeline stage.
- **#pattern** — Per-exchange DI scoping: the fix for "singleton non-thread-safe connection" is scoped lifetime, not thread-safety. The diagnosis is more valuable than the fix.

## Cross-References

- [[Databases and Data]] — hub for database architecture and storage patterns
- [[SQLite Is All You Need]] — SQLite as a production database backend (redb's SQLite provider makes this practical in .NET)
- [[Celly — Native .NET CEL Implementation]] — another Apache 2.0 .NET tool filling a gap in the ecosystem
- [[Queues Don't Fix Overload]] — the redb concurrency fixes are a case study in why queue consumers need correct parallelism, not just a semaphore
- [[Constraint Decay]] — structural constraints (like a compile-time default of 1 for dispatch concurrency) silently degrade system behavior, whether in coding agents or message brokers
- [[Open Source Agent Toolkit 2026]] — the broader open-source ecosystem map; redb fills the integration/data layer for .NET

---
*Sources: [[raw/redb-330-enterprise-net-stack]]*
*Last updated: 2026-07-18*
