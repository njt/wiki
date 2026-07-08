---
url: https://github.com/flare-db/flare-db
title: FlareDB
author: Ganesh Sivakumar
date_fetched: 2026-07-08
date_published: 2025
---

# FlareDB — Full Repo Analysis

## Overview

FlareDB is an Apache Beam-native streaming database written in Rust. It accepts Beam pipeline jobs (submitted via a Java SDK runner), fuses the pipeline graph into executable stages, and executes them either natively (for core transforms like Impulse and GroupByKey) or by delegating to a Java Beam SDK harness. PCollections are persisted in an embedded Arrow-native LSM-tree store (Tonbo).

Version: 0.1.8. License: Apache 2.0. Single author: Ganesh Sivakumar.

## Project Structure

```
flare-db/
  flaredb/             # core engine (Rust)
    src/
      main.rs          # entry: starts 6 gRPC servers on 127.0.0.1:8099
      lib.rs           # module declarations, DEFAULT_API_SERVICE_URL
      engine/
        mod.rs
        executor.rs    # StageExecutor: BFS traversal of executable graph
        store.rs       # FlareElementStore, BeamRecord, Arrow schema derivations
        coders.rs      # Beam wire-format coders (varint, KV, iterable, windowed value)
        harness/       # Beam Fn API servers
          control.rs   # FlareControlService: register/process bundle gRPC
          data.rs      # FlareDataService: element streaming with multiplexer
          log.rs       # FlareLogService
          state.rs     # FlareStateService
      fusion/
        fuser.rs       # GreedyPipelineFuser: greedy fusion algorithm
        pipeline.rs    # QueryablePipeline, ExecutableGraph, FusedPipeline
        stage.rs       # ExecutableStage, CollectionConsumers, SiblingKey
        refs.rs        # SideInputRef, TimerRef, UserStateRef
      transforms/
        mod.rs         # FlareTransform trait, from_urn() dispatch
        impluse.rs     # Impulse: produces single empty-byte[] element
        gbk.rs         # GroupByKey: in-memory HashMap-based grouping
      jobservice/
        server.rs      # FlareJobService: Beam Job API gRPC implementation
        job.rs         # Job, JobStore, fuse_pipeline()
        artifact.rs    # ArtifactStore
        urns.rs        # All Beam URN constants
      utils/
        errors.rs      # Error types
        macros.rs      # check_argument! macro
        path.rs        # Directory layout helpers
        visualization.rs # DOT graph output
  flare-cli/           # CLI tool (Rust)
    src/main.rs        # flare init/up/down commands
  beam-model-rs/       # Generated protobuf Rust types for Beam model
  runner-sdk/java/     # Java runner SDK (FlareRunner, FlarePipelineJob)
  beam/model/          # Apache Beam protobuf definitions (vendored)
  example/wordcount/   # WordCount example pipeline
```

Code size: flaredb/src ~4,000 lines of hand-written Rust; beam-model-rs ~17,000 lines of generated protobuf types; flare-cli ~630 lines total.

## Full Architecture

### Entry Point (main.rs)

`flare_up()` starts 6 gRPC servers on a single tonic `Server::builder()`:

1. **JobServiceServer** — Beam Job API: PrepareJob, RunJob (plus stubs for GetJobs, GetState, Cancel, etc.)
2. **ArtifactStagingServiceServer** — Receives staged pipeline artifacts
3. **BeamFnControlServer** — Register process bundle descriptors + send process bundle instructions
4. **BeamFnDataServer** — Bidirectional element streaming between harness and runner
5. **BeamFnLoggingServer** — SDK harness log forwarding
6. **BeamFnStateServer** — State API (stub, not yet implemented)

All on one port (`127.0.0.1:8099`) — a single-node deployment model.

### Pipeline Ingestion Flow

1. Java SDK `FlareRunner.run(pipeline)` translates Beam pipeline to proto, calls `PrepareJob` gRPC
2. `FlareJobService.prepare()` receives pipeline proto, calls `Job::new()` which:
   a. Builds a `QueryablePipeline` (petgraph-based graph of PTransform/PCollection nodes)
   b. Runs `GreedyPipelineFuser.fuse_pipeline()` to group transforms into `ExecutableStage`s
   c. Builds an `ExecutableGraph` (DAG of Worker nodes + Runner nodes with ConsumerMetaData edges)
3. `FlareJobService.run()` spawns a Java `FnHarness` process, waits for gRPC connection, then calls `StageExecutor.execute_pipeline()`

### StageExecutor (executor.rs)

Executes the `ExecutableGraph` in a BFS traversal:

```
Root (Runner: Impulse) → Worker (fused SDK stages) → Runner (GroupByKey) → ...
```

For **Worker nodes** (SDK stages):
1. Register the `ProcessBundleDescriptor` with the harness via control channel
2. Send `ProcessBundleRequest` to start the bundle
3. Spawn a background task that reads the input PCollection from the store, encodes elements through the Beam coder pipeline, and sends them to the harness via the data channel
4. Spawn a concurrent task that receives output elements from the harness, decodes them, and writes to the store
5. `tokio::select!` between harness response and decode task completion, with 60s timeout

For **Runner nodes** (native transforms):
1. Build a `ProcessBundleDescriptor` with the transform's spec, PCollections, coders, etc.
2. Register the bundle with the harness
3. Call `runner_transform.execute(ctx)` directly — reads from store, processes, writes back

### Pipeline Fusion (fuser.rs)

`GreedyPipelineFuser` — the most architecturally interesting module:

1. **Root discovery**: Finds all transforms with no inputs, separates into runner (unfused) and SDK (fusible candidates)
2. **Sibling grouping**: Consumers of the same PCollection are grouped by `(PCollection, Environment)` key. Within each key, sibling groups are formed where every member is mutually compatible
3. **Greedy stage fusion**: From each sibling group, a `GreedyStageFuser` grows a stage by walking forward through the pipeline graph, fusing compatible PCollections into the stage until it hits a materialization boundary
4. **Fusion boundaries** — a PCollection MUST materialize (write to store) when ANY consumer:
   - Has state or timers (ParDo with state_specs/timer_family_specs)
   - Has side inputs
   - Runs in a different environment
   - Is a runner-implemented transform (GroupByKey, Impulse)
   - Is a GBK, CreateView, or Splittable DoFn process element
5. **Deduplication**: After fusion, `ensure_single_producer()` detects PCollections with multiple producers and injects synthetic Flatten transforms to merge partial outputs

After fusion, the pipeline is a `FusedPipeline` with:
- `sdk_stages: IndexSet<ExecutableStage>` — fused SDK transforms
- `runner_stages: IndexSet<PTransformNode>` — runner-implemented transforms that break fusion

### Storage Engine (store.rs)

`FlareElementStore` is the core state layer:

- **Backend**: Tonbo, an embedded LSM-tree database (uses `fusio` for async I/O, `typed_arrow` for Arrow integration)
- **Schema management**: `FlareSchemaRegistry` (DashMap-backed) maps `pcollection_id → Arrow Schema`. Schema is auto-derived from the first batch of records via `derive_schema_from_records()`
- **Record model**: `BeamRecord` enum with four variants:
  - `PRIMITIVE(PrimitiveValue)` — String, Bytes, Int64, Bool, Void
  - `KV(BeamKV)` — key/value pairs (both primitives)
  - `GBK(BeamGbk)` — key + iterable value (post-GroupByKey)
  - `ITERABLE(IterableValue)` — list of primitives
- **Write path**: Records → Arrow RecordBatch (with UUID element IDs + pcollection_id columns + typed collection column) → Tonbo ingest
- **Read path**: Tonbo scan with filter `pcollection_id = <id>` → RecordBatches → `record_batch_to_beamrecords()`
- **PCollection isolation**: Each PCollection gets its own Tonbo DB directory (`{base_path}/{safe_pcollection_id}`), with DB instances cached in a DashMap

### Beam Coder System (coders.rs)

Hand-written Rust implementations of Beam's standard wire-format coders:

- **VarInt encoding**: Standard unsigned LEB128, with signed values cast to u64
- **Length-prefixed strings/bytes**: varint length + raw bytes
- **KV coder**: Key coder followed by value coder (no length prefix — relies on element boundaries from WindowedValue)
- **Iterable coder**: 4-byte big-endian count + elements, with unknown-length fallback (-1 count + chunked varint counts)
- **WindowedValue coder**: timestamp_millis (big-endian u64 with sign-bit flip) + global windows (zero-length payload per window) + pane info (bit-packed first byte + optional signed varint index) + element
- **GBK detection**: When `KvCoder`'s value component is an `IterableCoder`, it's automatically promoted to `GbkCoder`

### Harness Communication (harness/)

Each Beam Fn API endpoint follows the same pattern:

1. Server struct holds an `Arc<Inner>` with `outgoing: Mutex<Option<Receiver>>` and `incoming: Mutex<Option<Streaming>>`
2. When the harness connects, the gRPC handler takes the outgoing Receiver (wrapping it in `ReceiverStream`) and stores the incoming stream
3. A paired "channel" struct holds the sender end and provides typed methods (`send_elements`, `register_bundle`, etc.)

The **data channel** is the most complex: it has an `ElementStreamMultiplexer` that routes elements from the harness back to the correct stage consumer by `DataKey {instruction_id, transform_id}`. A background task (`stream_elements`) drains the tonic stream and fans out to per-key unbounded channels.

### Runner Transforms

**Impulse** (impluse.rs): Produces a single `PrimitiveValue::Bytes(vec![])` element. This is the Beam root transform — every pipeline starts here.

**GroupByKey** (gbk.rs): Scans the input PCollection (expecting KV records), builds a `HashMap<PrimitiveValue, IterableValue>` grouping values by key, and writes GBK records. Pure Rust, no harness involved. This is the critical optimization — avoiding a shuffle through the harness.

### CLI (flare-cli/src/main.rs)

Three commands:
- `flare init` — Downloads FlareDB binary + Java harness JAR from GitHub releases, creates `~/.flaredb/{bin,instances}`
- `flare up` — Spawns FlareDB process with instance UUID, writes state.json, polls for port 8099
- `flare down` — SIGTERM → wait → SIGKILL, removes state.json

### Java Runner SDK

`FlareRunner` extends Beam's `PipelineRunner<FlarePipelineJob>`:
1. Translates pipeline to proto via `PipelineTranslation.toProto()`
2. Sends `PrepareJobRequest` → gets `preparation_id` + staging token
3. Stages artifacts via Beam's `ArtifactStagingService`
4. Sends `RunJobRequest` → blocks until job completes

## Key Techniques

1. **Greedy fusion with deduplication**: The fusion algorithm mirrors the Beam portability framework's approach in Java but reimplemented in Rust with petgraph. The dedup pass injecting synthetic Flattens for multi-producer PCollections is a correctness mechanism often missed by naive fusion implementations.

2. **PCollection-as-database**: Unlike most Beam runners that hold PCollections in memory or on disk as opaque blobs, FlareDB stores each PCollection as a typed Arrow RecordBatch in an LSM-tree with schema derived from the actual element types. This means PCollections are queryable and survive beyond pipeline execution.

3. **Schema auto-derivation**: `derive_schema_from_records()` inspects the first batch of elements and determines the Arrow schema. It enforces type homogeneity across batches (e.g., no mixing String and Int64 elements in the same PCollection).

4. **Dual execution model**: Runner transforms execute directly in Rust (no serialization, no harness overhead), while SDK transforms go through the Java harness. The fusion algorithm treats runner transforms as materialization boundaries — SDK transforms fuse until they hit a runner transform, then materialize.

5. **WindowedValue wrapping**: All elements crossing the runner-SDK boundary are wrapped in `WindowedValue` (timestamp + windows + pane info). Currently all windows are GlobalWindow, but the codec infrastructure supports general windowing.

## Design Decisions & Trade-offs

### What they optimized for
- **Correctness of the Beam protocol**: The code is meticulous about Beam's Proto definitions, URN matching, and coder spec compliance
- **Simplicity of deployment**: Single binary, single port, single-node, database-per-PCollection
- **Rust safety**: No unsafe code visible in the hand-written portions

### What they sacrificed
- **Distribution**: v0.1.0 is single-node only. The pipeline executor is a single BFS loop — no parallelism, no sharding
- **Streaming**: Only bounded sources on Global Window in v0.1.0. The roadmap lists watermarks, event-time processing, and triggers as future work
- **Production readiness**: ~8 `todo!()` stubs in the JobService gRPC implementation (GetJobs, GetState, Cancel, Drain, etc.)
- **Error handling robustness**: The executor has a known correctness bug: if a node fails after being added to the `executed` set, the pipeline state is inconsistent

### Interesting trade-offs
- **Arrow + LSM-tree vs. in-memory**: Most runners keep PCollections in memory or as serialized bytes. FlareDB chose Arrow RecordBatches in an LSM-tree — good for persistence and queryability, at the cost of write/read latency
- **Rust engine + Java harness**: The engine is Rust but the SDK harness is Java (the standard Beam FnHarness). This requires shipping both binaries and coordinating a Java subprocess
- **Greedy fusion vs. cost-based**: The fusion algorithm is greedy (BFS, fuse everything compatible) rather than cost-based (e.g., consider data volume, parallelism). Simpler, but may produce suboptimal stage boundaries for large workloads

## Comparison to Related Systems

- **Apache Flink**: Uses its own DataStream/Table API, not Beam. Flink has mature state management, checkpointing, and distributed execution that FlareDB lacks in v0.1.
- **Apache Spark Structured Streaming**: Micro-batch model. Uses Spark's own API, not Beam. Much more mature but different execution paradigm.
- **Google Cloud Dataflow**: The reference Beam runner. Fully managed, distributed, streaming-native. FlareDB is to Dataflow what an embedded database is to a cloud data warehouse — smaller, local, self-contained.
- **Kafka Streams**: Also a streams-and-tables model, but tied to Kafka as the source-of-truth. FlareDB's PCollections as persistent tables echoes Kafka Streams' KTable, but FlareDB uses the Beam programming model.
- **RisingWave**: Another Rust streaming database, but uses PostgreSQL wire protocol as its interface. FlareDB uses Beam gRPC instead — a different audience.
- **Materialize**: SQL-based streaming materialized views on top of Kafka/Postgres. FlareDB is more of a pipeline execution engine than a materialized view maintainer.

## Dependencies

- `tonbo` (=0.4.0-a1): Embedded LSM-tree database with Arrow support
- `arrow-*` (57.3.1): Apache Arrow columnar format
- `tonic` (0.14.2): gRPC framework
- `tokio` (1.38): Async runtime
- `petgraph` (0.8.3): Graph data structures for pipeline DAG
- `prost` (0.14.3): Protobuf codec
- `dashmap` (6.1.0): Concurrent hash maps
- `beam-model-rs` (2.70.0): Generated Rust types for Beam protobufs
- `beam-sdks-java-harness-2.72.0-flare-bundled.jar`: Bundled Java SDK harness (external, downloaded by CLI)
