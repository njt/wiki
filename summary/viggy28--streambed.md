---
url: https://github.com/viggy28/streambed
title: Streambed — Postgres-to-Iceberg CDC Engine
author: viggy28
date_fetched: 2026-06-05
date_published: 2025
---

# Streambed — Full Repo Analysis

## Overview

Streambed is a Go-based CDC (Change Data Capture) engine that streams Postgres WAL changes via logical replication, writes Parquet files to S3, and commits Apache Iceberg metadata. It also embeds a DuckDB query server that speaks the Postgres wire protocol, so you can query Iceberg tables with `psql`.

**Size**: ~18K lines of Go across `cmd/streambed/`, `config/`, and `internal/` (wal, iceberg, parquet, pipeline, server, state, storage, resync, simtest). Plus ~3.6K lines of integration tests.

**Dependencies**: pglogrepl (WAL decoding), pgx (Postgres client), iceberg-go, parquet-go, go-duckdb, psql-wire (Postgres protocol), cobra (CLI), AWS SDK v2, go-sqlite3.

## Architecture

### Pipeline (single-goroutine design)

The core is `internal/pipeline/pipeline.go` — a single goroutine that:
1. Calls `pglogrepl.StartReplication()` to begin streaming WAL
2. In a tight loop: receives WAL messages (with deadline-based timeouts for flush/standby timers)
3. Decodes pgoutput binary protocol v1 messages via `internal/wal/decoder.go`
4. Routes to `iceberg.Writer.HandleEvent()` which buffers rows per table
5. Flushes when row count OR time threshold is reached
6. Sends standby status updates (with a carefully computed ACK position)

The ACK formula is the key correctness invariant:
- `ack = receivedLSN` when all buffers empty
- `ack = min(receivedLSN, pendingMinLSN - 1)` otherwise

This ensures the replication slot never advances past data still in memory buffers. On crash, the slot replays from the last acknowledged position.

### Writer + Buffer System

`internal/iceberg/writer.go` maintains per-table buffers. Each buffer holds:
- Raw rows (INSERTs/UPDATEs as full tuples)
- Delete keys (for equality-delete matching)
- A `deletedKeys` set that tracks net-deleted keys within a batch (used to suppress earlier INSERTs if a DELETE arrives for the same key later in the same batch)
- `FirstLSN` — the oldest unflushed LSN, used for the ACK watermark computation

On flush, two code paths:
- **Append-only** (no deletes): Write new Parquet file, commit as append snapshot
- **Copy-on-write** (has deletes): Read all existing Parquet files from S3, filter deleted rows, dedup new rows (last-write-wins within batch, net-deleted keys excluded), combine, write replacement Parquet, commit as overwrite snapshot

### Iceberg Catalog

`internal/iceberg/catalog.go` implements a filesystem-based Iceberg catalog. Key design decisions:
- Uses `version-hint.text` as the version pointer (standard Iceberg convention)
- Writes metadata as `vN.metadata.json` files
- Snapshots reference manifest lists → manifest files → data files stored as Avro
- No catalog server — just S3 objects
- Tracks `streambed.last_flush_lsn` in snapshot summaries for recovery
- Handles schema evolution (ADD/DROP/TYPE_CHANGE columns) by diffing old vs new RelationMessages
- Empty-table handling: `commitEmptyTable()` writes a metadata version with `current-snapshot-id = -1` and clears the snapshots array — this is needed because DuckDB's iceberg_scan falls back to the last snapshot if present even with -1

### WAL Decoder

`internal/wal/decoder.go` decodes pgoutput plugin v1 messages:
- `RelationMessage`: tracks table schemas (columns, types, replica identity keys)
- `InsertMessage`: full new row
- `UpdateMessage`: old key tuple + new full row
- `DeleteMessage`: old key tuple (REPLICA IDENTITY columns)
- `TruncateMessage`: list of relation IDs

Schema change detection: the decoder caches RelationMessages by relation ID. When a new RelationMessage arrives for a known table, `diffRelationColumns()` compares old vs new column lists to detect ADD, DROP, TYPE_CHANGE, and KEY_CHANGE. The writer handles schema evolution by flushing the old-schema buffer, evolving Iceberg metadata, then updating the buffer.

TOAST handling: unchanged TOAST columns are detected and a warning is logged. With REPLICA IDENTITY FULL, the old tuple provides TOAST values. Without it, values fall back to NULL.

### Parquet Builder

`internal/parquet/builder.go` converts pgoutput text-format values to typed Go values and writes Parquet files using `parquet-go`. Key details:
- Iceberg field IDs are stamped on Parquet columns (parquet.FieldID) so iceberg_scan can match columns to schema
- Type mapping: PG OIDs → Parquet types (bool→boolean, int2/4→int32, int8→int64, float4→float, float8→double, date→date days since epoch, timestamps→microseconds)
- Snappy compression
- Date parsing: days since Unix epoch
- Timestamp parsing: multiple format attempts (with/without microseconds, with/without timezone)

### State Store

`internal/state/store.go` uses SQLite (WAL mode) as a fast-startup cache. The authoritative source of truth for LSN position is Iceberg snapshot summaries. SQLite stores:
- `synced_tables`: registered tables with column count and `backfill_lsn`
- `replication_state`: unused table (legacy)

The backfill LSN mechanism handles resync overlap: after a `COPY`-based backfill, events from the main replication slot with WAL position ≤ backfill LSN are dropped to avoid duplicates.

### Query Server

`internal/server/server.go` runs an embedded DuckDB with Iceberg + httpfs extensions. It speaks the Postgres wire protocol via `psql-wire`. Key features:
- Periodic catalog refresh (30s) — discovers new tables and refreshes Iceberg views
- DuckDB fatal error recovery: if DuckDB's engine is corrupted by bad Iceberg data, the server re-opens DuckDB
- Type normalization: DuckDB Decimal → string (psql-wire can't encode them natively)
- DuckDB type → PG OID mapping for 13 types
- S3 credentials configured via environment variables with MinIO defaults fallback

### Resync (Backfill)

`internal/resync/resync.go` and `internal/wal/resync.go` implement a consistent-snapshot backfill:
1. Creates a TEMPORARY replication slot with `EXPORT_SNAPSHOT`
2. Opens a second connection, sets snapshot, runs `COPY table TO STDOUT`
3. Streams rows through the Parquet builder, batching into Iceberg snapshots
4. Records `backfill_lsn` in state so the CDC consumer can filter overlapping events

The temp slot is automatically dropped when the replication connection closes — no cleanup needed.

### Simulation Test Framework

The most impressive engineering in the repo: `internal/simtest/` implements a Jepsen-style continuous correctness test harness:
- **Runner** (`runner.go`): orchestrates workloads, oracle validation, chaos injection, and metrics collection
- **Oracle** (`oracle/oracle.go`): reads all rows from Postgres and Iceberg (via DuckDB), diffs by primary key, reports missing/extra/mismatch
- **Workloads** (`workload/`): pgbench-like schema + event generators
- **Chaos** (`chaos/chaos.go`): periodically kills the streambed process and pauses MinIO
- **Supervisor** (`supervisor/`): manages the streambed subprocess lifecycle
- **Metrics** (`metrics/`): JSONL metrics collection
- **Invariants** (`oracle/invariants.go`): self-consistency checks on Iceberg data alone

This is a serious testing investment — not just unit tests but a continuous, adversarial verification framework.

## Key Techniques

### 1. Conservative ACK with pending-min-LSN watermark

The replication slot ACK is held back to `min(pendingMinLSN) - 1` across all table buffers. This means the slot never advances past unflushed data. On crash, WAL replays from the last acknowledged position. This is the standard approach for exactly-once CDC, but the implementation is notably clean — a single function `computeAck()`.

### 2. In-buffer dedup with deletedKeys

Within a single buffer batch, if a key is INSERTed then DELETEd, both operations are tracked. The `deletedKeys` set suppresses the INSERT at flush time. This avoids writing rows that were immediately deleted — a common edge case in CDC systems that many implementations get wrong.

### 3. Copy-on-write deletes

Rather than writing position-delete files (which many Iceberg engines struggle with), streambed implements full CoW: reads existing Parquet data, filters deleted rows, combines with new inserts, writes a replacement file. This trades write amplification for query compatibility — any Iceberg reader (DuckDB, Spark, Trino) can query the result without understanding position deletes.

### 4. Dual-source LSN recovery

On startup, streambed reads LSN positions from two sources:
- The replication slot's `confirmed_flush_lsn` (Postgres's view)
- Iceberg snapshot summaries (`streambed.last_flush_lsn` per table)

It picks `max(slotLSN, minIcebergLSN)` as the start position. This handles the case where the slot advanced past a table that had no events (cold table) — the table's Iceberg LSN may be older than the slot position, which is expected and handled gracefully.

### 5. Schema evolution with pre-flush

When a schema change is detected (column ADD/DROP/TYPE_CHANGE), the writer:
1. Flushes the current buffer with the old schema (so no data is lost)
2. Evolves Iceberg metadata (new schema version, field IDs preserved)
3. Updates the buffer's column list

This is correct because Postgres guarantees no DML can be in-flight for a table during DDL — by the time we see the new RelationMessage, all existing rows have the old schema.

### 6. Backfill overlap suppression

During resync, a temp slot exports a consistent snapshot at LSN L. The COPY reads all rows visible at L. The main slot replays from where it left off. Events with WAL position ≤ L for the backfilled table are dropped via the `backfill_lsn` filter. This prevents double-counting rows that were both COPYed and replayed.

### 7. Jepsen-style simulation testing

The simtest framework continuously runs workloads while injecting chaos (kill streambed, pause MinIO) and validating that Postgres and Iceberg remain consistent. This is the kind of testing you'd expect from a database company, not a solo developer's side project.

## Design Trade-offs

### Optimized for: correctness and simplicity over throughput

- Single-goroutine pipeline (no channels, no backpressure complexity)
- CoW deletes (write amplification but query compatibility)
- No partitioning support (all data in one Iceberg partition)
- Snappy compression only (no Zstd, no column encoding tuning)

### Optimized for: embedded deployment over distributed scale

- Embedded DuckDB (no external query engine needed)
- SQLite state (no external database for metadata)
- Filesystem Iceberg catalog (no Hive/Glue/REST catalog dependency)
- Single binary (no microservices)

### Cost: CoW write amplification

Every UPDATE/DELETE requires reading ALL existing Parquet files for the table. For large tables, this is expensive. The trade-off is that query engines don't need to understand position-delete files. For append-heavy workloads (timeseries, event logs), this is fine. For update-heavy workloads (OLTP mirroring), it could become a bottleneck.

### Cost: No exactly-once across crashes without gaps

The ACK mechanism ensures at-least-once delivery. Dedup on restart (via LSN comparison) handles replay. But if the process crashes between writing Parquet to S3 and committing Iceberg metadata, the data file is orphaned on S3 (no snapshot references it). This is safe but wastes storage.

### Cost: Single-table CoW reads all files

There's no index on Parquet files by key range. Every CoW merge reads every data file. For tables with many historical snapshots, this could be slow — though icebeg's manifest pruning helps if partition evolution is added later.

## Core Abstractions

1. **RowEvent** (`internal/wal/types.go`): The universal event type — carries schema, table, columns, key columns, operation, values, old key, and WAL LSN. Every operation (INSERT/UPDATE/DELETE/TRUNCATE) maps to this single type.

2. **tableBuffer** (`internal/iceberg/writer.go`): Per-table mutable state — accumulated rows, delete keys, deleted key tracking, column metadata, field ID mapping, and LSN watermarks. The flush operation transforms this into Parquet + Iceberg commits.

3. **ObjectStorage interface** (`internal/storage/s3.go`): PutObject/GetObject/HeadObject/ListPrefix/DeleteObjects — a clean seam that enables fault injection (fault_s3.go, mem_s3.go) and testing without real S3.

4. **Catalog** (`internal/iceberg/catalog.go`): Manages the Iceberg metadata lifecycle — table creation, schema evolution, snapshot commits, manifest management, and LSN tracking. This is where the Iceberg spec compliance lives.

## File-by-File Summary

| File | Lines | Purpose |
|------|-------|---------|
| cmd/streambed/main.go | 794 | CLI entry, subcommands, reconnect loop |
| config/config.go | 141 | Env var + flag config loading |
| internal/pipeline/pipeline.go | 447 | Single-goroutine WAL consumer + writer |
| internal/iceberg/writer.go | 700 | Per-table buffering, CoW merge, flush |
| internal/iceberg/catalog.go | 937 | Iceberg metadata CRUD, manifest mgmt |
| internal/iceberg/schema.go | ~120 | PG OID → Iceberg type mapping |
| internal/iceberg/avro.go | ~200 | Avro encoding for Iceberg metadata |
| internal/wal/decoder.go | 332 | pgoutput message decoding |
| internal/wal/types.go | 162 | Core event types |
| internal/wal/slot.go | 164 | Replication slot management |
| internal/wal/resync.go | 204 | COPY-based backfill |
| internal/wal/copy_text.go | 230 | COPY text format parser |
| internal/parquet/builder.go | 206 | Parquet file construction |
| internal/parquet/reader.go | ~100 | Parquet file reading (for CoW) |
| internal/server/server.go | 392 | DuckDB + psql-wire query server |
| internal/server/catalog.go | ~200 | Table catalog for query server |
| internal/state/store.go | 207 | SQLite state persistence |
| internal/storage/s3.go | 179 | AWS S3 client with retry |
| internal/resync/resync.go | 230 | Resync orchestration |
| internal/simtest/runner.go | 416 | Simulation test orchestrator |
| internal/simtest/oracle/oracle.go | 228 | Row-level Postgres vs Iceberg diff |
| internal/simtest/chaos/chaos.go | 84 | Process kill + MinIO pause chaos |
| test/integration/integration_test.go | 1755 | Main integration test suite |

## Comparison to Similar Projects

- **Debezium**: Much more mature, Kafka-centric, broader connector ecosystem. Streambed is simpler (no Kafka dependency) but narrower (Postgres only, Iceberg only).
- **Airbyte/Fivetran**: ETL/ELT tools, not CDC. Batch-oriented, not WAL streaming.
- **PeerDB**: Postgres-to-data-warehouse CDC. Similar space but targets Snowflake/BigQuery. Streambed targets Iceberg + S3 directly.
- **pgcapture**: Another Postgres CDC to various sinks. Uses the same pgoutput plugin but different architecture (multi-process).

Streambed's distinguishing feature: it's a **single Go binary** that does CDC, storage, and query serving. No JVM, no Kafka, no Spark, no external catalog. This is unusual in the CDC space where most tools are either JVM-based (Debezium) or require a message broker.

## Surprising Design Choices

1. **No channels between pipeline stages** — the CLAUDE.md explicitly calls out "single goroutine reads WAL → decodes → buffers → flushes → acks. No channels between stages." This is counter to Go convention but simplifies ACK bookkeeping enormously.

2. **Iceberg metadata written as raw JSON** rather than using iceberg-go's metadata APIs. The catalog builds `tableMetadata` structs and marshals them directly. This gives full control over the metadata format but means the code must stay compatible with Iceberg spec details.

3. **DuckDB as the query engine** rather than a dedicated OLAP engine. This keeps the binary small and eliminates external dependencies, but DuckDB's Iceberg support is relatively young and has edge cases (e.g., the empty-snapshot handling).

4. **No partition evolution** — all data lands in one partition. This keeps the implementation simple but limits query performance on large tables.

## Verdict

Streambed is a well-engineered, opinionated CDC tool. The architecture is deliberately simple (single goroutine, CoW deletes, embedded DuckDB) and the testing investment is exceptional for a solo project (Jepsen-style simulation with chaos injection). The main limitations are CoW write amplification for update-heavy workloads and lack of partitioning. For append-heavy analytical workloads on Postgres, this is a compelling alternative to the Debezium/Kafka/Spark stack.
