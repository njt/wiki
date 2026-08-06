# Streambed

Postgres-to-Iceberg CDC engine in a single Go binary. Streams WAL changes via logical replication, writes Parquet files to S3, commits Iceberg metadata, and serves queries through an embedded DuckDB that speaks the Postgres wire protocol — so you connect with `psql`. No JVM, no Kafka, no Spark, no external catalog. Built by viggy28.

---

## Architecture

```
Postgres WAL ──▶ Decode (pgoutput v1) ──▶ Buffer (per-table) ──▶ Parquet ──▶ S3 ──▶ Iceberg Commit
                                                                                    │
                                                                          DuckDB ◀──┘ (psql-wire on :5433)
```

**Single-goroutine pipeline** (`internal/pipeline/pipeline.go`): reads WAL, decodes, buffers, flushes, ACKs — all in one loop. No channels between stages. This is unusual in Go but makes ACK bookkeeping trivial: the standby status update is computed from `min(pendingMinLSN)` across all per-table buffers, so the replication slot never advances past unflushed data.

**Four subcommands** via Cobra CLI:
- `sync` — main daemon: CDC + optional query server
- `query` — standalone DuckDB query server (no Postgres connection)
- `resync` — one-shot backfill via `COPY` under a consistent snapshot
- `cleanup` — delete S3 objects + state for a table (prep for resync)

**Recovery**: on startup, reads LSN from two sources — the replication slot's `confirmed_flush_lsn` and Iceberg snapshot summaries (`streambed.last_flush_lsn`). Picks `max(slotLSN, minIcebergLSN)`. On pipeline crash, reconnects with exponential backoff (up to 10 retries, 60s max), re-reads flush LSNs from Iceberg to avoid replays.

## Key Techniques

### Conservative ACK Watermark

```go
func computeAck(receivedLSN, pendingMinLSN pglogrepl.LSN) pglogrepl.LSN {
    if pendingMinLSN == 0 {
        return receivedLSN  // all buffers empty — ack everything
    }
    hold := pendingMinLSN - 1
    if hold < receivedLSN {
        return hold  // hold back to oldest unflushed data
    }
    return receivedLSN
}
```

This is the correctness invariant. Every per-table buffer tracks `FirstLSN` — the WAL position of its oldest unflushed event. The standby ACK is held back to `min(FirstLSN) - 1` across all tables. On crash, the slot replays from the last acknowledged position, so no data is lost.

### In-Buffer Dedup

Within a single flush batch, if a key is INSERTed then DELETEd (or UPDATEd multiple times), the writer tracks `deletedKeys` and uses `dedupRows()` to keep only the last version. Keys that were net-deleted are excluded entirely. This uses a length-prefixed binary encoding for collision-free key string building.

### Copy-on-Write Deletes

Rather than position-delete files (which many Iceberg readers struggle with), streambed does full CoW for UPDATEs/DELETEs:
1. Read all existing Parquet files from S3
2. Filter rows matching delete keys (O(N+M) via hash set)
3. Dedup new rows (last-write-wins)
4. Combine surviving existing + deduped new → write replacement Parquet
5. Commit as Iceberg "overwrite" snapshot (replaces previous manifest, doesn't carry forward old data files)

Trade-off: write amplification on every update, but universal query compatibility.

### Schema Evolution with Pre-Flush

When a DDL change is detected (RelationMessage with different column list), the writer:
1. Flushes buffered rows with the old schema
2. Evolves Iceberg metadata (ADD/DROP/TYPE_CHANGE columns)
3. Looks up Postgres column defaults for ADD columns (via a second pgx connection)
4. Updates buffer's column list + field ID mapping

Also handles **schema drift on restart**: if DDL happened while streambed was down, the writer diffs WAL columns against Iceberg schema on first flush and evolves accordingly.

### Resync with Overlap Suppression

The `resync` command backfills a table via `COPY` under a consistent snapshot:
1. Creates a TEMPORARY replication slot with `EXPORT_SNAPSHOT` → gets (snapshot S, LSN L)
2. On a second connection: `SET TRANSACTION SNAPSHOT S` → `COPY table TO STDOUT`
3. Streams rows → Parquet → S3 → Iceberg snapshots (batched by `--flush-rows`)
4. Records `backfill_lsn = L` in SQLite state
5. On next `sync` startup, events with WAL position ≤ L for this table are dropped

The temp slot auto-drops when its connection closes — no cleanup needed.

### Embedded Query Server

DuckDB + Iceberg extension + httpfs extension, exposed over Postgres wire protocol (`psql-wire`). You connect with `psql -p 5433` and run SQL against Iceberg tables. DuckDB reads directly from S3. If the Iceberg table is corrupt and DuckDB fatally errors, the server re-opens DuckDB and retries view registration.

## Design Decisions

| Decision | Why | Trade-off |
|----------|-----|-----------|
| Single goroutine, no channels | Simplifies ACK bookkeeping; no backpressure bugs | No parallelism within a pipeline |
| CoW deletes (not position deletes) | Universal query compatibility | Write amplification for updates |
| Filesystem Iceberg catalog (S3) | Zero dependencies, works with MinIO | No concurrent writer safety |
| Embedded DuckDB for queries | Single binary, no external services | DuckDB Iceberg support is young |
| SQLite for state (not Iceberg-only) | Fast startup cache | Two sources of truth to reconcile |
| Avro manifests (not JSON) | Iceberg spec compliance | Extra encoding/decoding code |
| Snappy compression only | Fast, good enough for most data | No Zstd/LZ4 options |
| No partitioning | Implementation simplicity | No partition pruning on queries |

## Simulation Testing

The most impressive part of this codebase is `internal/simtest/` — a Jepsen-style continuous correctness harness:

- **Workloads**: pgbench-like schema with INSERT/UPDATE/DELETE generators
- **Oracle**: reads all rows from Postgres AND Iceberg (via DuckDB), diffs by primary key, reports missing/extra/mismatch
- **Chaos**: periodically kills the streambed process + pauses MinIO to exercise recovery paths
- **Metrics**: JSONL output with replication lag, oracle results, restart counts

This is database-company-level testing for a solo project. The chaos module (`internal/simtest/chaos/chaos.go`) is just 84 lines but covers the two most important failure modes: process death and storage unavailability.

## Comparison

- **vs Debezium**: Debezium is Kafka-centric, JVM-based, multi-connector. Streambed is a single Go binary targeting only Postgres→Iceberg. Debezium has a broader ecosystem; Streambed has dramatically lower operational complexity.
- **vs PeerDB**: Both do Postgres CDC. PeerDB targets data warehouses (Snowflake, BigQuery). Streambed targets Iceberg on S3 with an embedded query engine. PeerDB is a company; Streambed is a solo project.
- **vs pgcapture**: Similar technical approach (pgoutput plugin). pgcapture has multiple sink types. Streambed is Iceberg-only but has the embedded query server and simulation testing.
- **vs Airbyte/Fivetran**: These are batch ETL/ELT, not CDC. Different latency profile. Streambed has sub-second latency (streaming WAL).

Streambed is a practical instantiation of the log-centric data integration architecture Jay Kreps laid out in [[The Log — Unifying Abstraction for Real-Time Data]]: the database's WAL is the log, the CDC pipeline is the subscriber that reads and transforms, and Iceberg on S3 is the destination that other systems can consume independently. The entire architecture — single-source-of-truth log, decoupled consumers, each reading at their own pace — is Kreps' blueprint implemented in a single Go binary.

Streambed's niche: **you want Postgres CDC to Iceberg on S3, queriable with `psql`, and you want it as a single binary with no infrastructure dependencies.** That's a narrow but real need — and nothing else fills it as simply.

---

*Source: [[summary/viggy28--streambed]]*
*Tags: #tool #project #database #cdc #iceberg #postgres*
*Last updated: 2026-06-05*
