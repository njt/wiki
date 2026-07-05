---
url: https://github.com/documentdb/documentdb
title: DocumentDB — MongoDB-Compatible Open Source Document Database on PostgreSQL
author: Microsoft
date_fetched: 2026-07-05
date_published: 2025
---

# Raw Analysis: DocumentDB (GitHub Repository)

Repository: https://github.com/documentdb/documentdb
License: MIT

## Project Overview

DocumentDB is an open-source MongoDB-compatible document database built entirely on PostgreSQL. It is NOT a standalone database engine — it's a wire-protocol compatibility layer that lets any MongoDB client (pymongo, mongosh, drivers) talk to PostgreSQL as if it were MongoDB. The project is produced by Microsoft and is one of the most sophisticated examples of "make an existing database speak a different protocol" in the open-source world.

## Repository Structure

The project has four main components:

1. **pg_documentdb_core** — C PostgreSQL extension introducing the BSON data type
2. **pg_documentdb** — C PostgreSQL extension with the full DocumentDB API (~241K lines of C)
3. **pg_documentdb_gw** — Rust gateway translating MongoDB wire protocol to PostgreSQL queries (~272 .rs files)
4. **pg_documentdb_gw_host** — Rust pgrx extension embedding the gateway as a PostgreSQL bgworker

Plus:
- **pg_documentdb_extended_rum** — Extended RUM index for full-text search
- **internal/pg_documentdb_distributed** — Distributed/shared extensions (likely Citus integration)
- **pg_documentdb_gw/libs/tokio-postgres** — Vendored and patched tokio-postgres

## Architecture Analysis

### Three-Layer Architecture

#### Layer 1: BSON Type Extension (pg_documentdb_core)

`pg_documentdb_core/src/pg_documentdb_core.c` registers a native `bson` type in PostgreSQL. The key trick is the libbson memory vtable integration: each dynamically-linked extension (.so file) gets its own static copy of libbson. The `InstallBsonMemVTablesLocal()` function routes libbson allocations through PostgreSQL's `palloc` family, meaning BSON data is managed by PostgreSQL memory contexts and benefits from automatic cleanup on context reset. The extension MUST be loaded via `shared_preload_libraries` — it refuses to load otherwise.

The control file specifies `requires = 'documentdb_core, pg_cron, tsm_system_rows, vector, postgis'`, showing the dependency chain: vector search (pgvector), geospatial (PostGIS), full-text (RUM), and cron scheduling are all PostgreSQL extensions that DocumentDB layers on top of.

#### Layer 2: API Implementation (pg_documentdb)

This is the bulk of the project. The C code implements MongoDB operations as PostgreSQL-callable functions:

- `aggregation/bson_aggregation_pipeline.c` (10,933 lines) — The aggregation pipeline engine. Each stage of a MongoDB aggregation ($match, $group, $project, $unwind, $bucket, $densify, $lookup, etc.) is compiled into PostgreSQL queries operating on BSON values.
- `aggregation/bson_aggregates.c` (2,719 lines) — Aggregate operators ($sum, $avg, $min, $max, $first, $last, $push, $addToSet)
- `aggregation/bson_query.c` (481 lines) — Query builder translating MongoDB filter documents to PostgreSQL WHERE clauses on BSON
- `aggregation/bson_projection_tree.c` (380 lines) — Projection handling for field inclusion/exclusion
- `aggregation/bson_aggregation_search.c` — Full-text search via RUM index
- `aggregation/bson_positional_query.c` — `$[]`, `$[<identifier>]` positional operators
- `aggregation/bson_unwind.c` — `$unwind` array decomposition
- `aggregation/bson_bucket_auto.c` — `$bucketAuto` with automatic boundary computation
- `auth/scram256.c` — SCRAM-SHA-256 authentication
- `src/api_hooks.c` — Plugin architecture using hook function pointers

The hook architecture (`api_hooks.c`) is particularly interesting. ~20 function pointer hooks (`IsMetadataCoordinator_HookType`, `RunCommandOnMetadataCoordinator_HookType`, `DistributePostgresTable_HookType`, etc.) allow the distributed extension to inject sharding behavior. When `pg_documentdb_distributed` is not loaded, these hooks remain NULL and single-node behavior applies. This is a clean plugin pattern in C.

#### Layer 3: Wire Protocol Gateway (pg_documentdb_gw)

The Rust gateway is the entry point for MongoDB clients. Architecture:

```
TCP/Unix Socket → [TLS Detection/Handshake] → Connection Loop → Message Parser → Request Dispatch → SQL Execution → Response Writer
```

**Connection handling** (`runtime/v1.rs`):
- `tokio::select!` across IPv4, IPv6, and Unix socket listeners
- TLS detection via `peek()` at the first 3 bytes (checking for TLS handshake 0x16 0x03)
- Per-connection `BufStream` with configurable read/write buffer sizes
- TCP keepalive (180s time, 60s interval), TCP_NODELAY
- Each connection spawns a Tokio task

**Connection loop** (`service/connection_loop/stream_driver.rs`):
- Read-ahead: starts reading the next message header while the current request is being processed
- Message pipelining: MongoDB OP_MSG allows multiple in-flight requests; the loop handles this correctly
- Error reply: if a read failure occurs, writes an error response; if the write fails, terminates the connection
- Writer shutdown with 1-second timeout

**Protocol module** (`protocol/`):
- `message.rs` (387 lines) — OP_MSG parsing with BSON section detection, checksum support, and flag bit validation
- `reader.rs` (1,072 lines) — Low-level sync reads from a byte cursor, handling BSON document boundaries and section types
- `bson_scanner.rs`, `bson_writer.rs` — BSON-level scanning and response serialization
- `opcode.rs` — MongoDB wire protocol opcodes (OP_MSG, OP_INSERT, OP_QUERY, OP_REPLY, etc.)
- Constants: 16 MB max BSON object, 48 MB max message, 250 KB max pre-auth message

**Request dispatch** (`processor/process.rs`):
A massive match statement dispatching ~50+ MongoDB command types to handler functions:
- Data management: find, insert, update, delete, aggregate, count, distinct, validate, collStats, dbStats, currentOp, killOp
- Data description: create, drop, dropDatabase, collMod, renameCollection, shardCollection, unshardCollection
- Indexing: createIndexes, dropIndexes, listIndexes, reIndex
- Auth: createUser, updateUser, dropUser, usersInfo, createRole, updateRole, dropRole, rolesInfo
- Transaction: commitTransaction, abortTransaction, prepareTransaction
- Cursor: getMore, killCursors
- Session: endSessions, killSessions

Transaction handling wraps the entire request: before processing, `transaction::handle()` checks if a transaction needs to be created or resumed. After processing, if the connection is in a transaction and the result was a WriteConflict or Find/Aggregate error, the transaction is automatically aborted.

**Query catalog** (`postgres/query_catalog.rs`):
All SQL sent to PostgreSQL is centralized here. The Rust gateway never builds SQL dynamically — it substitutes parameters into pre-defined templates. For example:
```rust
insert: "SELECT * FROM documentdb_api.insert($1, $2, $3, NULL)"
find_cursor_first_page: "SELECT cursorPage, continuation, persistConnection, cursorId FROM documentdb_api.find_cursor_first_page($1, $2)"
```

**Connection pooling** (`postgres/conn_mgmt/`):
- `PoolManager` maintains three tiers of pools:
  - `system_requests_pool` (max 2 connections) — admin queries
  - `system_auth_pool` (max 5 connections) — authentication queries
  - `user_data_pools` (DashMap keyed by `(username, PgPoolSettings)`) — per-user data access
  - `shared_data_pools` — shared across users
- Pool cleanup runs every 300 seconds, disposes pools unused for 7200 seconds
- `ScopedTransaction` guard handles transaction lifecycle: starts READ COMMITTED if no transaction exists, auto-rolls back on drop if not committed

**Data client** (`postgres/documentdb_data_client.rs`, 1,459 lines):
`DocumentDBDataClient` implements the `PgDataClient` trait with methods for every operation. Each method:
1. Gets the SQL template from `QueryCatalog`
2. Binds parameters (db name, collection name, BSON document)
3. Runs the query through the connection pool
4. Transforms PostgreSQL rows into MongoDB-compatible responses

The `execute_aggregate` method shows the pattern:
```rust
self.run_db_bson_cursor(
    request_context,
    connection_context,
    self.service_context.query_catalog().aggregate_cursor_first_page(),
    QueryOptions::builder().supports_backend_timeout(true).build(),
)
```

**Explain diagnostics** (`explain/`):
The `query_diagnostics.rs` module parses PostgreSQL EXPLAIN output using regexes to extract: index usage, scan types (DocumentDBApiScan, DocumentDBApiQueryScan, DocumentDBApiCursorScan), filter conditions, sort keys, and output counts. This is how MongoDB's `explain()` command is implemented — it runs PostgreSQL's EXPLAIN on the generated query and re-interprets the plan in MongoDB terms.

**Telemetry** (`telemetry/`):
- OpenTelemetry traces with context propagation
- SQLCommenter: injects trace context into SQL comments for Postgres log correlation
- `RequestTracker` for per-request metrics
- `TelemetryManager` wraps both tracer and meter providers

### Layer 4: Background Worker (pg_documentdb_gw_host)
Embedded PostgreSQL background worker via pgrx, running the gateway inside the PostgreSQL process rather than as a separate daemon. Uses Cargo.toml workspace configuration.

## Key Techniques

### BSON as a First-Class PostgreSQL Type
The foundational technique: create a `bson` type that PostgreSQL understands, with operators, casts, and functions. This means MongoDB documents are stored as BSON values in regular PostgreSQL tables. The C extension provides functions like `bson_build_document()`, `bson_array_agg()`, `bson_dollar_project()`, etc. that operate on BSON within SQL. A collection is essentially `CREATE VIEW ... AS SELECT document FROM ...`.

### Two-Level Query Compilation
MongoDB query → BSON filter tree → PostgreSQL SQL with BSON operators → PostgreSQL execution plan. The aggregation pipeline (`bson_aggregation_pipeline.c`) translates each stage into SQL operations. The explain feature re-parses the generated plan to show users meaningful diagnostics.

### Libbson Memory Vtable Trick
Each extension .so links its own static libbson. The vtable setup routes BSON allocations through PostgreSQL's `palloc`, so BSON objects respect PostgreSQL memory contexts. This is why DocumentDB extensions must be preloaded — the vtable needs to be set before any BSON allocation.

### SQL Query Catalog Pattern
The `QueryCatalog` struct in Rust maps every operation to a SQL template string. This is unusual — most proxies build SQL dynamically. The trade-off: less flexibility (can't dynamically change query structure per-request), but much cleaner boundaries and easier audit of what SQL is actually being sent.

### Hook Architecture for Optional Distribution
The C extension exposes ~20 function pointers for distribution behavior. When the distributed extension is loaded, it populates these hooks. When not loaded, they're NULL and single-node behavior applies. This is a well-executed plugin pattern in C.

### Read-Ahead Pipelining
The connection loop starts reading the next message header via `tokio::spawn` while the current request is being processed. This means the next message's header is already available when the current one finishes, enabling near-zero overhead pipelining.

## Design Trade-Offs

### Optimized For
1. **Operational simplicity**: You get PostgreSQL's backup, replication, monitoring, and security infrastructure for free
2. **Correctness**: All operations run through PostgreSQL's transaction system — there's no custom storage engine to get wrong
3. **Extensibility**: The hook architecture and extension model make it possible to add distribution/sharding without touching core code
4. **Compatibility**: The wire protocol compatibility is extensive — ~50+ MongoDB commands supported, including aggregation pipeline, indexes, RBAC, sessions, and transactions

### Sacrificed
1. **Performance vs. native MongoDB**: Every MongoDB operation becomes a PostgreSQL function call, which means function call overhead, parameter binding, and plan caching overhead. For simple key-value lookups, this is slower than MongoDB's native storage engine
2. **Dynamic query flexibility**: The SQL catalog pattern means queries can't be dynamically composed at the gateway level. Complex queries must be handled entirely in the C functions
3. **Memory overhead**: Each extension .so carries its own static libbson (~1MB+ per .so), and BSON document overhead vs. JSONB
4. **Development complexity**: The project requires expertise in C (PostgreSQL internals), Rust (async networking), MongoDB wire protocol, BSON format, and PostgreSQL extension development

### Compared to FerretDB
FerretDB is another MongoDB-to-PostgreSQL proxy, but stores documents as JSONB. DocumentDB's native BSON type means:
- **Better type fidelity**: BSON preserves types that JSON/JSONB loses (dates, binary data, ObjectId, Decimal128, regex)
- **Better index support**: PostgreSQL can build type-aware indexes on BSON values
- **More complex**: Requires C extensions and preload, vs. FerretDB's single Go binary
- **More complete**: Supports aggregation pipeline stages that FerretDB hasn't implemented yet

## Code Statistics

- ~241,000 lines of C (pg_documentdb)
- ~272 .rs files (pg_documentdb_gw)
- ~10,933 lines in aggregation pipeline alone
- ~50+ MongoDB commands supported
- 4 extension .so files (core, api, extended_rum, distributed)
- 1 Rust gateway binary
- Multi-language: C, Rust, SQL, Python (tests)
