# DocumentDB

Microsoft's open-source MongoDB-compatible document database built entirely on PostgreSQL. Rather than a standalone database engine, it's a **wire-protocol translation layer**: any MongoDB client (pymongo, mongosh, drivers) connects to a Rust gateway that speaks the MongoDB wire protocol, which translates operations into PostgreSQL function calls operating on a native `bson` data type. The result: you get MongoDB's document API backed by PostgreSQL's 30 years of durability, replication, backup, and security infrastructure. MIT licensed.

---

## Architecture

DocumentDB is a four-layer stack, each layer building on the one below.

### Layer 1: BSON Type Extension (`pg_documentdb_core`)

A C PostgreSQL extension that registers a native `bson` data type. The key mechanism: each dynamically-linked extension .so carries its own static copy of libbson, and a memory vtable routes BSON allocations through PostgreSQL's `palloc`. This means BSON data respects PostgreSQL memory contexts — freed automatically on context reset, safe from memory leaks. The extension must be loaded via `shared_preload_libraries`.

`pg_documentdb_core/src/pg_documentdb_core.c:31-35` — `DocumentDBCore_InstallBsonMemVTablesLocal()` sets the allocation vtable.

### Layer 2: DocumentDB API (`pg_documentdb`, ~241K lines of C)

The operational layer. Every MongoDB operation is implemented as a PostgreSQL-callable C function:

```
SELECT documentdb_api.insert($1, $2, $3, NULL);
SELECT documentdb_api.find_cursor_first_page($1, $2);
CALL documentdb_api.update_txn_proc($1, $2, $3, NULL);
```

The crown jewel is `aggregation/bson_aggregation_pipeline.c` (10,933 lines) — a query compiler that translates MongoDB's aggregation stages ($match, $group, $project, $unwind, $bucket, $densify, $lookup) into PostgreSQL SQL operating on BSON values. Each stage becomes a SQL operation; the pipeline's execution runs inside PostgreSQL's planner and executor.

`pg_documentdb/src/api_hooks.c` exposes ~20 function pointers for optional distributed/sharding behavior. When the internal `pg_documentdb_distributed` extension loads, it fills these hooks. When absent, hooks stay NULL and single-node behavior applies. This is a clean plugin pattern: distribution concerns are injected, not baked in.

Depends on: pg_cron, tsm_system_rows, vector (pgvector), postgis.

### Layer 3: Wire Protocol Gateway (`pg_documentdb_gw`, 272 Rust source files)

The entry point that MongoDB clients connect to. Architecture:

```
TCP/Unix Socket → TLS Handshake → Connection Loop → Message Parser → Request Dispatch → SQL → Response
```

**Connection lifecycle** (`runtime/v1.rs`):
- `tokio::select!` across IPv4, IPv6, and Unix socket listeners
- TLS detection via `peek()` at 3 bytes (detecting 0x16 0x03 handshake)
- Per-connection `BufStream` with configurable buffers
- TCP keepalive (180s/60s) and TCP_NODELAY

**Message pipeline** (`service/connection_loop/stream_driver.rs`):
Uses read-ahead: spawns the next header read while processing the current request. This means the next message header is already available when the current one finishes — zero-latency pipelining for MongoDB's OP_MSG protocol.

**Protocol** (`protocol/message.rs`):
Full OP_MSG parsing with BSON sections, checksum support, flag bit validation. Constants: 16 MB max BSON object, 48 MB max message, 250 KB pre-auth limit.

**Request dispatch** (`processor/process.rs`):
A match statement routing ~50+ MongoDB commands to handlers. Covers data management (find, insert, update, delete, aggregate, count, distinct), data description (create/drop/rename collections/databases, sharding), indexing (createIndexes, dropIndexes, reIndex), RBAC (users, roles), transactions, cursors, sessions, and cluster operations (currentOp, killOp).

**The key design pattern**: the `QueryCatalog` (`postgres/query_catalog.rs:549-661`) maps every operation to a SQL template string. The gateway never constructs SQL dynamically — it only substitutes parameters:
```rust
// From query_catalog.rs
insert: "SELECT * FROM documentdb_api.insert($1, $2, $3, NULL)"
find_cursor_first_page: "SELECT cursorPage, continuation, persistConnection, cursorId
                         FROM documentdb_api.find_cursor_first_page($1, $2)"
```

**Connection pools** (`postgres/conn_mgmt/pool_manager.rs`):
Three-tier pooling with DashMap-based isolation:
- System requests pool (max 2 connections): admin queries
- Auth pool (max 5): SCRAM-SHA-256 authentication
- Per-user data pools keyed by `(username, PgPoolSettings)`: user data access
Pools unused for 7200 seconds are disposed; cleanup runs every 300 seconds.

**Connection management** (`postgres/scoped_transaction.rs`):
`ScopedTransaction` is a RAII guard: starts READ COMMITTED if no transaction exists, auto-rolls back on drop if not committed. This is used per-SQL-query to ensure each operation either commits or rolls back cleanly.

**Explain** (`explain/query_diagnostics.rs`):
Runs PostgreSQL's `EXPLAIN` on generated queries, then parses the JSON output using regexes to extract scan types (DocumentDBApiScan, DocumentDBApiQueryScan, DocumentDBApiCursorScan), index usage, and filter conditions — surface back to users as MongoDB-compatible explain output.

**Telemetry**: OpenTelemetry traces with SQLCommenter (injects trace context into SQL comments for Postgres log correlation). Per-request metrics via `RequestTracker`.

### Layer 4: Background Worker (`pg_documentdb_gw_host`)

Embeds the Rust gateway inside PostgreSQL's process space via pgrx. An alternative to running the gateway as a standalone daemon.

---

## Key Techniques

### 1. Native BSON Type in PostgreSQL

The foundational technique. Rather than storing documents as JSONB (like FerretDB), DocumentDB makes BSON a first-class PostgreSQL type with its own operators, casts, and functions. This preserves type fidelity (dates, binary, ObjectId, Decimal128, regex) that JSONB loses, and enables type-aware indexing. The libbson memory vtable trick (`InstallBsonMemVTablesLocal`) routes allocations through `palloc` so BSON objects participate in PostgreSQL memory lifecycle.

### 2. Two-Level Query Compilation

MongoDB aggregation pipeline → BSON filter tree → PostgreSQL SQL with BSON operators → PostgreSQL execution plan. The pipeline compiler (`bson_aggregation_pipeline.c`) translates each stage into SQL; the planner then optimizes; the explain feature re-parses the plan to surface meaningful diagnostics. This is not just translation — it's compilation: the pipeline stages define what to compute, and PostgreSQL decides how.

### 3. Centralized SQL Catalog

Every SQL statement the gateway sends is defined in `QueryCatalog`. Unlike other proxies that build SQL dynamically, DocumentDB centralizes all SQL in a single struct. This means:
- SQL changes only happen in one place
- The C API surface is the contract; the Rust gateway is a thin routing layer
- Explain/diagnostics parsing can match against known SQL patterns

### 4. Hook-Based Distribution

The C code exposes ~20 function pointers (`api_hooks.c`) for distribution behavior. When `pg_documentdb_distributed` loads, it fills these. When absent, behavior is single-node. This is a clean plugin pattern achieved in C, avoiding #ifdef hell.

### 5. Read-Ahead Pipelining

The connection loop reads the next message header concurrently with request processing. MongoDB OP_MSG supports multiple in-flight requests per connection; DocumentDB's read-ahead ensures they're processed with minimal latency overhead.

---

## Design Decisions

### What DocumentDB Optimizes For

**Operational simplicity above raw performance.** By running on PostgreSQL, DocumentDB inherits decades of operational infrastructure: replication, backup, PITR, monitoring, upgrades, security. You don't need to learn a new storage engine's failure modes.

**Protocol compatibility over storage innovation.** DocumentDB doesn't try to build a better document store. It makes PostgreSQL accept MongoDB's wire protocol. All innovation is in the translation and BSON type layers.

**Correctness through PostgreSQL transactions.** Every write goes through PostgreSQL's transaction system. There's no custom write-ahead log, no custom replication protocol. The trade-off: function call overhead on every operation.

### What DocumentDB Sacrifices

**Raw performance vs. native MongoDB.** Every MongoDB operation is at minimum one PostgreSQL function call. Simple key-value lookups will be slower than MongoDB's native mmap'd storage engine. The aggregation pipeline runs inside PostgreSQL, which means plan caching, executor overhead, and BSON serialization costs.

**Flexibility of dynamic SQL.** The SQL catalog pattern means the gateway can't dynamically compose queries. If a new MongoDB feature requires a structurally different SQL query, both the C function and the SQL template must be updated.

**Deployment complexity.** You need PostgreSQL plus four C extensions plus a Rust binary. FerretDB ships as a single Go binary. The BSON type requires `shared_preload_libraries` configuration — it can't be loaded on the fly.

### Compared to FerretDB

| Dimension | DocumentDB | FerretDB |
|---|---|---|
| Document storage | Native BSON type | JSONB |
| Language | C + Rust | Go |
| Wire protocol | Full OP_MSG | Subset of OP_MSG |
| Aggregation pipeline | 10K-line compiler in C | Limited SQL generation in Go |
| Index support | Type-aware BSON indexes | JSONB GIN/GiST indexes |
| Deployment | 4 C extensions + Rust binary | Single Go binary |
| Type fidelity | Complete (Date, Binary, ObjectId, Decimal128) | Partial (JSON types) |

The BSON-native approach means DocumentDB has higher type fidelity and index quality at the cost of deployment complexity.

---

## Comparison Notes

Unlike [[Streambed]] which streams PostgreSQL changes *out* to other systems, DocumentDB streams MongoDB protocol *into* PostgreSQL. Both are "make PostgreSQL be something else" projects but in opposite directions.

Unlike [[Artie]] which replicates data between databases, DocumentDB is a translation layer — data lives in PostgreSQL, and MongoDB clients access it through the gateway.

The hook architecture in `api_hooks.c` follows the same plugin pattern seen in PostgreSQL itself (hook-based extensibility), but applied to document database distribution. Similar to how [[PgDog]] uses hooks for connection pool routing.

The centralized query catalog pattern is reminiscent of [[Snorkel]]'s labeling functions or a data pipeline approach — define the operations declaratively, let the engine execute them.

---

Tags: #tool #project #database #nosql #postgresql #mongodb #rust #c

*Sources: [[raw/documentdb]]*
*Last updated: 2026-07-05*
