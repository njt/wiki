---
url: https://dev.to/rinat_kozin/redb-330-an-enterprise-net-stack-you-actually-own-typed-store-a-homegrown-apache-camel-and-3gd1
title: "redb 3.3.0: an enterprise .NET stack you actually own — typed store, a homegrown Apache Camel, and a runtime with a dashboard (all free)"
author: Rinat Kozin
date_fetched: 2026-07-18
date_published: 2026-07-09
---

# redb 3.3.0: an enterprise .NET stack you actually own — typed store, a homegrown Apache Camel, and a runtime with a dashboard (all free)

**Author:** Rinat Kozin (creator of the redb ecosystem)
**Published:** July 9, 2026 on dev.to
**Tags:** #dotnet #opensource #llm #sqlite
**GitHub:** github.com/redbase-app | **Site:** redb.ru

---

## The Three Layers

1. **redb** — A typed store for .NET. Write a POCO, tag it with an attribute, use full LINQ server-side with no migrations and no `Include`. Providers: Postgres, MSSQL, SQLite. Free and Pro editions (change tracking, bulk, cache, analytics).

2. **redb.Route** — An integration engine in the Apache Camel spirit: route DSL (`From(...)...To(...)`), 30+ connectors (Kafka, RabbitMQ, HTTP, gRPC, S3, LLM, etc.), enterprise integration patterns, transactions, telemetry.

3. **redb.Tsak** — A runtime that turns routes into a production service: dashboard, hot-reloadable modules (`.tpkg`), live context management, metrics, cluster with coordinator and failover. Ships as NuGet packages, Docker images, and standalone archives.

### Basic redb Usage

```csharp
[RedbScheme]
public class Note
{
    public string Tag  { get; set; } = "";
    public string Text { get; set; } = "";
}

// write
await redb.SaveAsync(new RedbObject<Note> { Props = new() { Tag = "work", Text = "hello" } });

// read — server-side query
var notes = await redb.Query<Note>()
    .Where(n => n.Tag == "work")
    .ToListAsync();
```

Schema comes from `SyncSchemeAsync<Note>()` — generated off the class itself, no migration files.

---

## redb.Route — What's New in 3.3.0

### Two New Transports

#### Amazon SQS/SNS (`redb.Route.Sqs`)

One package, two schemes: `sqs://` (queue) and `sns://` (topic). Built on the native AWS SDK for .NET v4.

```csharp
// Consumer
From(Sqs.Queue("orders").WaitTimeSeconds(20).ConcurrentConsumers(5))
    .Process(ex => Handle(ex.In.Body));

// Publisher
From("timer://tick?period=5000")
    .SetBody(_ => BuildEvent())
    .To(Sns.Topic("order-events"));
```

Features: long-polling, real competing consumer loops, visibility timeout extension, transactional ack via `.Transacted()`, FIFO support, SNS→SQS auto-subscription. Compatible with LocalStack/ElasticMQ. Full AWS credential chain honored; W3C trace context across hops.

#### Telegram (`redb.Route.Telegram`)

The `telegram://` scheme, built on `Telegram.Bot`. Consumer is a long-poller (single `getUpdates` stream per token); producer covers `send`/`document`/`photo`/`edit`/`delete`/`answer`.

```csharp
// Echo bot
From(Tg.Receive(token))
    .Process(ex => ex.Out.Body = $"You said: {ex.In.Body}")
    .To(Tg.Send(token).ChatId(Header(TelegramHeaders.ChatId)));
```

Features: honors 429 `retry_after` contract, validates `parseMode`, supports inline/reply keyboards, webhook-unpack pipeline (`UnpackTelegramUpdate`), fluent DSL. Delivery is **at-most-once** (Telegram advances update offset upon handing it to you). Parallelism via `.Threads(N)`.

---

### RAG, End to End: `knowledge://`, `embed://`, and `.Knowledge()`

Three new pieces that close the RAG loop:

**`knowledge://` (ingest scheme):** Takes message body as a document, chops into chunks (deterministic character windows with overlap), embeds each if an `IEmbeddingProvider` is registered, upserts into `IKnowledgeStore`.

```csharp
From("file://docs?include=*.md").To("knowledge://handbook");
```

Chunk IDs follow `{docId}#{index}` format — re-ingesting replaces chunks in place.

**`embed://` (embeddings step):** The mirror of `llm://` — turns message body text (or batch) into a vector on `Out.Body`. URI host names a connection factory.

**`.Knowledge(collection, k)` (retrieval DSL step):** Pulls top-K chunks for current message and "injects them into the system prompt" so the next `.To("llm://…")` answers grounded on them.

```csharp
From("kafka://questions")
    .Knowledge("handbook", k: 5)
    .To("llm://claude");
```

Semantic search when `IEmbeddingProvider` is present (cosine `SearchAsync`), keyword otherwise (`SearchTextAsync` via server-side `LIKE`). No store wired = no-op, never breaks pipeline.

**`knowledge_search` (ready-made tool):** An `.AsLlmTool` route — input `{query, top_k?, collection?}` → `{results:[…]}`. Collection can be **pinned** to fence an agent.

**Notable fix:** Switched to `UnsafeRelaxedJsonEscaping` across LLM serializers — previously non-ASCII characters were escaped as `\uXXXX`, roughly 6×'ing tokens for Cyrillic/CJK content and breaking keyword `LIKE` matching in the knowledge store.

---

### `.Threads(N)` — Concurrency on Any Source

A processing-concurrency stage that caps a route section at N parallel workers — no named `seda://` endpoint required.

```csharp
From("mqtt://sensors")
    .Threads(8)
        .Process(ex => HeavyWork(ex))
    .EndThreads();
```

Adaptive to exchange pattern: for **InOnly** (fire-and-forget) it's a hand-off with clone + worker pool; for **InOut** it runs inline on the same exchange under a `SemaphoreSlim` gate so the reply survives. Supports `.MaxQueueSize(n)` and `.EnqueueTimeout(TimeSpan)`. Ordering not preserved at N > 1.

---

### The Big Fix: `ConcurrentConsumers(N)` Finally Works

> "`ConcurrentConsumers(N)` was silently giving you no parallelism on the brokers."

The option existed and sized a `SemaphoreSlim` — but no actual concurrency resulted.

**Per connector:**

- **RabbitMQ:** Channel was created with `CreateChannelOptions(...)` leaving `consumerDispatchConcurrency` at its compile-time default of `1` (in RabbitMQ.Client 7.2.1), which overrode the connection-level setting. Now channels open with explicit dispatch concurrency matching `ConcurrentConsumers`. (Fix in 3.2.2.)

- **Kafka:** New `EnableAutoCommit` option (default `true`). Fix for transacted producer throwing `Local: Erroneous state` on deferred send. (3.2.1.)

- **AMQP 1.0 and IBM MQ:** Serial receive loop `await`ed `Process` inline before pulling next message — `SemaphoreSlim` was dead. Now N genuine competing consumers each with own session/connection. IBM MQ fix also addresses connection-scoped syncpoint: "Backout() rolls back the whole connection, so each worker must own its own." Topics clamped to single subscriber with warning. (3.2.1.)

**Upgrade warning:** If a route had `ConcurrentConsumers(N > 1)` and "worked," it was serial. After upgrade it genuinely parallelizes — "per-queue ordering is no longer preserved" and handlers must be thread-safe. Routes at default `1` are untouched.

---

### Per-Exchange Connections — End of the Captive Singleton

`IRedbService` wraps one non-thread-safe DB connection. DSL steps like `ProcessWithRedb(...)`, `SetBodyFromRedb(...)`, `SetHeaderFromRedb(...)`, and `BeginRedbTransaction()` previously used a "single `IRedbService` captured from the root DI container." Under real concurrency, two exchanges drove that one connection and the driver threw errors like "A command is already in progress."

Now each exchange gets its "own DI scope → its own scoped `IRedbService` → its own pooled connection," cached and disposed with the exchange. Also added `controller.Redb()` for controllers.

**Additional fixes:** ThreadsProcessor/SedaProducer/VmProducer leaked clones on failed hand-off; WireTapProcessor leaked tap clone when callback threw; scheduled `llm://` and `exec://` consumers never disposed per-tick exchange; lazy producer start-up made thread-safe.

Author states: 3.3.0 is "the release after which you can actually push redb.Route hard and stop chasing flaky connection errors."

---

### Putting It Together: RAG Bot on Telegram

**Ingest route:**
```csharp
From("file://docs?include=*.md")
    .To("knowledge://handbook?chunkChars=1000&overlap=100&embed=true");
```

**Question → answer route:**
```csharp
From(Tg.Receive(token))
    .Threads(4)
        .Knowledge("handbook", k: 5)
        .To("llm://claude")
    .EndThreads()
    .To(Tg.Send(token).ChatId(Header(TelegramHeaders.ChatId)));
```

**Tool-based alternative (model decides when to search):**
```csharp
context.AddRoutes(new KnowledgeSearchTool(new KnowledgeSearchOptions {
    Collection        = "handbook",
    EmbeddingProvider = embeddings
}));

From(Tg.Receive(token))
    .Threads(4)
        .To("llm://claude?tools=knowledge_search")
    .EndThreads()
    .To(Tg.Send(token).ChatId(Header(TelegramHeaders.ChatId)));
```

---

## redb (the DB Core) — What Made It In

1. **Fail-fast guard on provider connection** — If same `IRedbService` entered from two threads, throws a clear `InvalidOperationException` naming the cause. Uses cheap `Interlocked` check.

2. **Query parser fix for `array.Contains(x)` in `WhereRedb` on .NET 9 / C# 13** — `string[]` `.Contains(x)` inside a predicate was binding to `ReadOnlySpan` overload (`MemoryExtensions.Contains`) instead of `Enumerable.Contains`, causing `NotSupportedException`. Parser now recognizes and translates to `IN` clause.

3. **`ComputeHash()` NRE fix** — On object with `Props == null`, the path dereferenced before null check. Now returns `null` (→ `Guid.Empty`) consistently.

4. **Connection-pool leak fix** — A throw from driver's transaction `DisposeAsync()` skipped disposing `_connection`, preventing return to pool. Under failure bursts the leak drained the pool. Now uses `try/finally` on both connection and transaction wrapper.

---

## redb.Tsak (the Runtime) — What Made It In

1. **New connectors bundled** — `redb.Route.Sqs` and `redb.Route.Telegram` ship in the distribution.
2. **Dashboard scheduler shows cron jobs without `Quartz` config section** — Tsak now always hands out one shared `IScheduler` (falling back to in-memory `RAMJobStore`). `AdoJobStore` still used for persistence across nodes.
3. **Users admin API uses per-request scoped `IRedbService`** — `UsersController` previously used shared singleton; now per-request via `controller.Redb()`.
4. **From 3.2.1:** Endpoints page no longer hides anonymous contexts it counted in stats; standalone web archive starts on port 8080 instead of 5000.

**Shipping:** NuGet packages, Docker images (`redb-tsak-{worker,web,stack}` on .NET 9, tags `:3.3.0-net9`, `:3.3.0`, `:latest`) on `ghcr.io/redbase-app`, cosign-signed, plus standalone archives with `checksums.txt` and `.bundle` signatures.

---

## About Pro Being Free

> "No catch." Pro remains proprietary (closed source) but across the 3.x line it is "handed out for free and with no bookkeeping — no licenses, keys, sign-up, license server, or node/volume limits."

**What was previously paid, now free:**
- redb core: change tracking, bulk ops, advanced cache, analytical queries (grouping, window functions)
- Tsak: cluster with coordinator, distributed lock, leader election, failover

"dotnet add package and you're working."

---

## Upgrading from 3.2.x — Checklist

1. **Routes with `ConcurrentConsumers(N > 1)`** — now genuinely parallelize; ensure thread-safe handlers. If order matters, keep at `1`.
2. **Shared state in handlers** — anything that worked because route was effectively single-threaded can now run from N threads.
3. **`IRedbService` in routes** — no action needed; per-exchange connections auto-enable. Old connection-in-progress errors should disappear.
4. **Version alignment** — everything at `3.3.0`; older 3.2.x connectors are API-compatible but lack concurrency fixes.
5. **Tsak: rebuild or pull fresh image** — new connectors only land in shared layer after rebuild.

---

## Getting It

```bash
# NuGet packages (core and providers)
dotnet add package redb.Core
dotnet add package redb.Postgres      # or redb.MSSql / redb.SQLite
dotnet add package redb.Postgres.Pro  # Pro — free, no key

# Engine and connectors
dotnet add package redb.Route
dotnet add package redb.Route.Sqs
dotnet add package redb.Route.Telegram
dotnet add package redb.Route.Llm

# Runtime (Docker)
docker pull ghcr.io/redbase-app/redb-tsak-stack:3.3.0

# Verify signature
cosign verify --key cosign.pub ghcr.io/redbase-app/redb-tsak-worker:3.3.0
```

---

## Series (earlier posts)

1. Leaving MassTransit for a Camel state of mind: the Kafka connector, Scatter-Gather, and transactions
2. Apache Camel for .NET, dissected: the HTTP connector with no ASP.NET MVC + the Content-Based Router
3. redb.Route — Apache Camel for .NET: 22 transports, 30+ EIP patterns, compiled DSL

---

## Key Claims

- The stack is "one coherent ecosystem" rather than a "menagerie" of separately-vended components
- Pro packages are genuinely free with "no catch" — no license server, keys, sign-up, or limits
- The RAG loop works "with zero external dependencies beyond the model" — no separate vector store needed
- `ConcurrentConsumers(N)` fix is the headline maturity improvement — it was "quietly broken under load"
- The connection-pool leak fix prevents a scenario requiring restart
- Images are cosign-signed for supply chain security
