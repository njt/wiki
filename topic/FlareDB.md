# FlareDB

An Apache Beam-native streaming database written in Rust that treats data pipelines and persistent storage as a single unified system. Beam PCollections materialize into queryable Arrow-backed tables — the boundary between "processing" and "storing" dissolves.

---

## Architecture

FlareDB is a single-binary gRPC server (`127.0.0.1:8099`) that speaks the Apache Beam Job API and Beam Fn API. A Java SDK runner submits pipeline protos; the Rust engine fuses transforms into stages and executes them either natively or by spawning a Java FnHarness subprocess.

**Dual execution model**: Core transforms like `Impulse` and `GroupByKey` run directly in Rust (`flaredb/src/transforms/impluse.rs:52`, `flaredb/src/transforms/gbk.rs:50`). SDK transforms (ParDo, etc.) execute in the Java harness. The pipeline fusion algorithm treats runner transforms as materialization boundaries — SDK stages fuse greedily until they hit one.

**Storage layer**: Each PCollection is an Arrow RecordBatch persisted in a Tonbo LSM-tree (`flaredb/src/engine/store.rs:822`). Schema is auto-derived from the first batch of elements. This means PCollections survive pipeline completion and are queryable — a deliberate departure from in-memory-only Beam runners.

**Pipeline ingestion path**:
1. Java `FlareRunner` translates pipeline → proto, calls `PrepareJob` + `RunJob`
2. `Job::new()` builds a `QueryablePipeline` graph (`flaredb/src/fusion/pipeline.rs:404`), then runs greedy fusion → `FusedPipeline` → `ExecutableGraph`
3. `StageExecutor.execute_pipeline()` (BFS, `flaredb/src/engine/executor.rs:78`) walks the DAG, for each node either dispatching to the harness or calling `runner_transform.execute()`

**Fusion algorithm** (`flaredb/src/fusion/fuser.rs:29`): Groups consumers of the same PCollection by `(PCollection, Environment)` into sibling sets, then greedily extends each stage forward until hitting a boundary (state/timers, side inputs, environment change, runner transform). A subsequent deduplication pass injects synthetic `Flatten` transforms when multiple producers feed the same PCollection.

## Key Techniques

### Greedy pipeline fusion with dedup
The fuser reimplements the Beam portability framework's fusion logic in Rust, using `petgraph` for the pipeline graph. The dedup pass (`flaredb/src/fusion/fuser.rs:727`, `ensure_single_producer()`) handles the case where fusion creates multiple producers for one PCollection by rewriting each to emit a *partial* PCollection and injecting a Flatten — a correctness requirement that naive fusion implementations often miss.

### PCollection-as-database
Unlike most Beam runners that keep PCollections in memory or as opaque blobs, FlareDB stores each as typed Arrow RecordBatches in an LSM-tree. `derive_schema_from_records()` (`flaredb/src/engine/store.rs:437`) auto-derives the Arrow schema from the first batch, enforcing type homogeneity. This turns pipeline intermediate state into durable, queryable tables.

### Hand-written Beam wire coders
The coder system (`flaredb/src/engine/coders.rs`) implements Beam's standard wire format in Rust: LEB128 varints, length-prefixed strings/bytes, KV with implicit boundaries, iterables with both known-length and chunked unknown-length modes, and WindowedValue with pane info bit-packing. Notably, when a `KvCoder`'s value component is an `IterableCoder`, it auto-promotes to `GbkCoder` (`flaredb/src/engine/coders.rs:89`).

### Element stream multiplexing
The data channel (`flaredb/src/engine/harness/data.rs:91`) uses an `ElementStreamMultiplexer` backed by `DashMap<DataKey, UnboundedSender>` to route harness output to the correct stage consumer. A single background task drains the gRPC stream and fans elements out by `{instruction_id, transform_id}` — avoiding N×M gRPC connections.

## Design Decisions

**Single-node by design (v0.1.0)**: No sharding, no distribution, no parallelism beyond tokio tasks. The executor is a single-threaded BFS loop. This is either a deliberate "get the protocol right first" choice or an early-stage limitation — the code doesn't comment on it.

**Rust engine + Java harness split**: The engine is Rust for safety and performance; the SDK harness is Java because Beam's standard `FnHarness` is JVM-only. This requires shipping both binaries and managing a Java subprocess lifecycle — a deployment complexity trade-off for protocol compatibility.

**Arrow + LSM-tree over in-memory**: Most runners keep PCollections in memory for speed. FlareDB chose durability and queryability via Arrow RecordBatches in Tonbo. The trade is write/read latency for persistence — a bet that "streaming database" means data should survive the pipeline.

**Greedy over cost-based fusion**: The fuser fuses everything mutually compatible, without considering data volume, parallelism, or cluster topology. Simpler to implement and reason about, but may produce suboptimal stage boundaries at scale.

**Known gaps**: ~8 `todo!()` stubs in JobService (GetJobs, GetState, Cancel, Drain, etc.), no state/timer implementation, bounded sources only on Global Window, and a flagged correctness bug in the executor where a node marked as `executed` before completion produces inconsistent state on failure.

## Comparison Notes

Unlike [[Apache Flink]] which provides its own DataStream/Table API, FlareDB uses Apache Beam as both programming model and execution protocol. This gives it a standard API but limits it to Beam's abstraction level.

Unlike [[Streambed]] and [[Artie]] (CDC pipelines), FlareDB is a pipeline *execution engine*, not a data movement tool. It runs transforms, not just copies data.

Unlike [[Materialize]] (SQL materialized views over Kafka), FlareDB uses the Beam programming model rather than SQL, and targets pipeline authors rather than analytics consumers.

The streams-and-tables unification echoes [[Kafka Streams]]' KTable concept and the [[Lakebase and LTAP]] vision of a single storage layer serving both operational and analytical workloads, but FlareDB does it within the Beam ecosystem.

Unlike most [[Databases and Data]] projects that separate compute from storage, FlareDB's "PCollection as Arrow table" design makes every pipeline intermediate a database table — the boundary between processing and storing is deliberately dissolved.

The harness-communication pattern (background task draining gRPC stream, per-key channel fan-out) resembles the multiplexing patterns seen in [[Components of a Coding Agent]] and [[Loop Engineering]], where independent work streams share a single transport.

---

*Sources: [[raw/flare-db]]*
*Last updated: 2026-07-08*
