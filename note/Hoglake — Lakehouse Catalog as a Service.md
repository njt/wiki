# Hoglake — Lakehouse Catalog as a Service

Hoglake is PostHog's open lakehouse catalog: an open table format where the metadata (snapshots, schemas, file statistics, deletion vectors) lives as rows in Postgres behind a REST service, and the data lives as parquet in object storage. Its thesis is that DuckLake had the right idea (RDBMS metadata) but the wrong deployment (a client-embedded library); hoglake makes the catalog a *service* with one implementation, one version, one deployment. The evidence it is a serious system: ~66K lines of Kotlin server, a deployed production incarnation (Gigahog in dev, prod-us, prod-eu), and a benchmark harness that doubles as a regression guard.

---

## Architecture

Eight components in one repo, all speaking one wire contract (`server/src/main/resources/openapi/hoglake.yaml`, explicitly the single source of truth):

- **server/** — the control plane: Kotlin, Ktor, JDBI, Flyway, Postgres 18. ~66K lines. Routes map one-to-one to the OpenAPI spec (`api/Routes.kt`).
- **pyhoglake/** — Python client that owns the parquet *writer* path: it writes the files itself (field ids stamped into the parquet schema), extracts footer statistics, and registers files in one commit.
- **duckdb-client/** — a C++ DuckDB extension that speaks the REST contract (`ATTACH 'hoglake:my_catalog'`). DuckDB as a client, never as the engine the catalog lives inside.
- **server/trino/** — harness for a native Trino connector kept in PostHog's Trino fork.
- **hedgerow/** — replication daemon: one process, one source table, one destination, append-only, at-least-once. Its offset lives in the *source* catalog's changefeed, so it carries no cursor tables.
- **millrace/** — Kafka→hoglake ingestion with durable SlateDB staging on object storage; Kafka offsets commit only after rows are durable in staging.
- **webui/**, **bench/** — React console and a benchmark harness whose scenarios assert catalog invariants after load and fail loudly rather than report numbers they can't stand behind.

The schema (`server/schema.sql`, frozen at v1.0 and append-only) encodes the design: versioned-row pattern (`begin_snapshot <= S AND (end_snapshot IS NULL OR S < end_snapshot)` for time travel), an identity/version split (`hog_table` immutable identity with a `table_uuid` incarnation, `hog_table_version` carries mutable name/namespace), per-catalog id allocators advanced inside the commit tail so snapshot ids are dense and ordered with commit order, and a typed change vocabulary (`hog_snapshot_change` with a CHECKed `kind` enum) replacing comma-encoded change strings.

## Key techniques

- **The two-statement commit tail.** `commit/CommitTail.kt` is the most interesting file. A one-file prepared append previously sent 19 SQL statements under the per-catalog lock — each a network round trip paid while every other writer queued. Production data showed ~67 commits/s of small commits pushing p50 commit latency to ~30s. The fix: one READ_SQL and one WRITE_SQL, with all the Kotlin checks between them. Why not one statement: under READ COMMITTED a statement's snapshot is taken when it *starts*, so the lock-taking statement would read pre-wait state; and the typed refusals would need a second implementation in PL/pgSQL. Every CTE in the write hangs off the allocator UPDATE, guarded by the head and next-file-id the read returned.
- **Typed conflict vocabulary.** Three refusals share one recovery: 409 `ddl_since_read_snapshot`, 409 `table_recreated`, 410 below the expiry floor. Bodies say `retry: re-prepare` — none is retryable by replay. Appends never conflict with appends; a delete declares its `read_snapshot` and conflicts with any insert/DDL on touched tables since. Conflict detection is one indexed lookup on `hog_snapshot_change`.
- **Footer-shipping registration.** The server never opens a data file on the commit path. Writers send footer statistics; a deferred-stats mode has the `hydrator/Hydrator.kt` sweep read footers from object store asynchronously, and the same read doubles as a field-id audit.
- **Server-assigned row lineage.** Contiguous row-id ranges, never reused, under a `table_uuid` incarnation contract; compaction preserves row ids across a sorted rewrite (`compaction/ExternalMergeSort.kt` runs a spill-budgeted external merge sort — the spill bound is a typed exception because an overrun gets the pod killed by kubelet, not an IOException).
- **Bounds in mapped-Iceberg encoding.** Column statistics are stored in Iceberg single-value serialization of the *mapped Iceberg type*, never the hoglake type, so manifest generation for the Iceberg facade is a mechanical copy. Details like canonicalizing signed zeros (Iceberg's natural-order comparison would read a `−0.0`/`+0.0` pair as an empty range and prune a file containing 0.0) are pinned cross-language in test vectors.
- **Fuzzing and parity.** The 27-type column vocabulary is enforced identically by the `ColType` enum, a Postgres CHECK, and the OpenAPI enum, with `ScalarTypeParityTest` asserting them against each other from their actual files; nightly fuzz CI and property tests.

## Design decisions

- **Postgres as the only backend, unapologetically.** Foreign keys with `ON DELETE CASCADE`, partial indexes, advisory locks are load-bearing. The cost: one backend choice, one service to operate — a deliberate anti-Iceberg trade. Iceberg buys ubiquity with file-based metadata and pointer-swap commits; hoglake buys indexed-SQL metadata operations and one place where commit, compaction and expiry are implemented.
- **One honest gap:** blind partitioned appends are unguarded until a server flag flips (`refuseBlindPartitionedAppends`), because the DuckDB client sends exactly that shape today — the rule is an invariant, the flag only delays when it bites.
- **Auth is opt-in and currently off.** ES256 bearer tokens, per-catalog scopes, STS credential vending via a sibling `hogtower` service — designed, documented, but `HOGLAKE_AUTH_MODE=off` is the default. For a system this central, that is a known open edge.
- **Benchmarks as guards, not bragging.** Every performance claim in the README is a scenario in `bench/` with invariant assertions after load, run as regression tests.

## Comparison notes

- [[DuckLake — SQL as a Lakehouse Format]]: hoglake's direct ancestor and foil. DuckLake keeps the catalog in a client-embedded extension — every client commits through the database directly, so commit paths race client-side and maintenance is client-invoked procedures without locks. Hoglake's answer is the service boundary itself; the repo even maintains a "ducklake defect ledger" documenting the predecessor's production defects and where each is answered.
- [[Iceberg Table Cleanup]]: Iceberg's metadata is files in object storage fetched layer by layer (metadata.json → manifest list → manifests); hoglake's is indexed SQL scoped to the write set. Iceberg remains the ubiquity play — hoglake is designing an Iceberg REST facade (`docs/iceberg-federation.md`) rather than fighting for engine support.
- [[Open Table Formats (Databricks)]]: hoglake is a fourth point on the Delta/Iceberg/DuckLake map, notable for putting *maintenance* (compaction, expiry, cleanup) under server-side advisory locks instead of racing client procedures.
- [[Postgres Transactions Are a Distributed Systems Superpower]]: this project is a maximalist case study for that argument — the entire serialization story is one advisory lock plus FKs, partial indexes and versioned rows, and the commit-tail rewrite shows both the power and the round-trip cost of leaning on Postgres hard.

---
*Sources: [[raw/hoglake]], [[summary/hoglake]]*
*Last updated: 2026-11-06*
