---
url: https://github.com/antflydb/antfly
title: AntFly
author: AJ Roetker and contributors
date_fetched: 2026-07-11
date_published: 2025
---

# AntFly

AntFly is a distributed search engine and AI-native database built on etcd's Raft
consensus library. It combines full-text search (BM25), vector similarity (HBC trees,
RaBitQ quantization), and graph traversal over multimodal data — text, images, audio,
and video. Embeddings, chunking, and graph edges are generated automatically as data
is written. Built-in RAG agents, an inference runtime, and a web dashboard tie it
all together.

## Architecture Summary

AntFly uses a **multi-Raft** design with separate consensus groups for metadata
(cluster topology, table schemas, shard assignments) and storage (one Raft group per
shard). The system spans multiple languages:

- **Go** (`go/pkg/antfly/`): The primary server. Handles HTTP API, metadata orchestration,
  Raft consensus, distributed coordination, and the index subsystem. Uses Pebble
  (CockroachDB's RocksDB successor) as its storage engine and bleve for full-text indexing.
  ~100K+ lines.
- **Zig** (`zig/`): A full rewrite of the local shard data plane. Replaces the Go Pebble/bleve
  stack on each shard with LMDB (via mmap) or a pure-Zig LSM tree. Also contains the
  inference runtime (`pkg/inference/`) for embeddings, chunking, reranking, and generation.
  The Zig engine communicates with Go via a stable C ABI (`libantfly`).
- **TypeScript** (`ts/`): SDK, React component library (`@antfly/components`), the Antfarm
  web dashboard, and a graph visualization package.
- **Rust** (`rs/`): SDK binding, plus the pgaf PostgreSQL extension that exposes AntFly
  search via a custom index access method using the `@@@` operator.
- **Python** (`py/`): SDK and packaging scripts.

### Multi-Raft Design

Each shard has its own Raft consensus group. The transport layer uses QUIC (HTTP/3)
for reduced connection establishment and head-of-line blocking avoidance. The
`MultiRaft` type (`go/pkg/antfly/src/raft/multiraft.go`) manages all active shard
Raft groups on a node, using a `singleflight.Group` to prevent duplicate startup
races.

### Data Flow

1. Documents are written via the HTTP API → metadata server routes to the owning shard
2. The shard's Raft leader proposes the write to its consensus group
3. On commit, the state machine writes to Pebble (Go) or LMDB/LSM (Zig)
4. Derived work (embeddings, summaries, chunks, graph edges) is appended to a
   sequence-based derived log
5. Per-index workers advance watermarks from that log, applying enrichments asynchronously

### Index System

Uses a registry pattern (`go/pkg/antfly/src/store/indexes/indexes.go`). All indexes
implement the `Index` interface; enrichable indexes add `EnrichableIndex` for async
embedding generation. Index types include `full_text` (BM25 via bleve), `embeddings`
(vector via RaBitQ-quantized HBC trees), `graph` (relationship traversal), `remote`
(proxy), and `algebraic` (sparse-token sidecar).

## Key Techniques

### HBC Dense Indexing

The Hierarchical Balanced Clustering (HBC) tree is a disk-backed approximate nearest
neighbor index. Key design choices:
- **Write routing** uses exact child-centroid distance at each internal node — NOT
  quantized error-bound competitive sets. Quantized error bounds are for search only.
- **Bulk build** constructs final nodes once, quantizes once per finished node (not once
  per inserted vector), and persists vectors and metadata before tree construction.
- Several bulk-build strategies coexist: recursive, Hilbert-seeded, and doc-key-seeded.

### RaBitQ Quantization

Vector quantization using RaBitQ (a rotation-based binary quantization method).
Enables configurable precision/performance tradeoffs for approximate nearest neighbor
search.

### Algebraic Sparse-Token Database Theory

A research thread (`zig/ALGEBRAIC.md`, ~16K lines of design notes) that models the
database as a sparse formal vector space:

```
database state = sparse vector / formal sum of tuples
query plan = composition of algebraic transforms
aggregation = fold into a monoid, group, semiring, or lattice
updates = delta vectors
```

Each document fact is tokenized into basis coordinates. Queries compile into algebraic
operators (filter, project, fold). Aggregates declare their algebraic structure —
COUNT/SUM use additive groups (invertible, good for incremental maintenance), MIN/MAX
use semilattices (monotone but non-invertible), and joins use semiring-like
contractions over shared dimensions.

The implemented prototype (`pkg/antfly/src/storage/db/algebraic/`) materializes
algebraic tokens as a sidecar index alongside full-text, vector, and graph indexes.
It supports COUNT, SUM, AVG, MIN/MAX, temporal cylinders (hour/day/month buckets),
configured composite equi-joins, and adaptive materialization (the system observes
query shapes and auto-creates materialized tensor expressions when warranted).

### Shard Split Without Full Rebuild

Shard splitting optimizes for minimal data movement:
- **Child docstore**: Page-level LMDB copy — clone right-hand subtrees page-for-page,
  rebuild only mixed branch spine and split leaf
- **Text indexes**: Segment handoff — classify segments by key range metadata, copy
  right-only segments unchanged, rebuild only mixed segments
- **Dense indexes**: HBC subtree handoff — classify nodes by `min_doc_key`/`max_doc_key`,
  hand off right-only subtrees, rebuild only mixed subtrees
- **Sparse indexes**: Postings block handoff — copy right-only posting chunks raw,
  rebuild only mixed blocks
- **Graph**: Rebuilds reverse state directly from owned outgoing edge keys, no generic
  doc replay

### Document Identity (DOCID) System

A comprehensive document identity architecture (`zig/DOCID.md`):
- **Binary component codec**: Order-preserving escape encoding so arbitrary byte
  sequences (including NUL, 0xFF, delimiter-shaped bytes) can be document IDs
- **Two-layer identity**: `canonical_doc_id` (deterministic per shard) + `doc_ordinal`
  (compact `u32` for Roaring bitmap efficiency)
- **ResolvedDocSet**: Internal query representation with three forms — `doc_keys` for
  tiny sets, sorted `ordinals` for small/medium sets, `ordinal_bitmap` for dense sets
- **Ordinal-backed filters**: All index families (full-text, dense, sparse, algebraic,
  graph) converge on `doc_ordinal` as the common document identity for cross-index
  filtering

### Simulation-Based Testing

The `go/pkg/antfly/src/sim/` package contains a full deterministic simulation harness:
- In-memory network with configurable faults (partition, delay, drop)
- Mock clock for deterministic replay
- Random and scenario-based test drivers
- Jepsen-inspired chaos testing for distributed protocols
- This complements the TLA+ formal specs

### TLA+ Formal Verification

Four protocols are formally specified and model-checked:
- `AntflyTransaction.tla` — distributed transaction protocol
- `occ-2pc.tla` — optimistic concurrency control with two-phase commit
- `AntflySnapshotTransfer.tla` — Raft snapshot transfer
- `AntflyShardSplit.tla` — shard split coordination

### Go/Zig Dual-Stack Architecture

The production system is a two-language hybrid:
- **Go owns distribution**: Raft consensus, metadata management, HTTP API, cluster
  orchestration, and the `DB` interface
- **Zig owns the local data plane**: Storage engine (LMDB/LSM), all index types,
  query execution, the algebraic sidecar, and the inference runtime
- They communicate via a stable C ABI (`zig/CAPI.md`) with typed binary codecs for
  hot paths (dense kNN, simple full-text) to avoid JSON overhead across the boundary
- The migration rule: keep the Go `DB` interface stable, move the hot local data plane
  into Zig, keep distributed orchestration in Go

## Design Decisions

### Optimized for Write-Time Enrichment

Rather than requiring users to pre-compute embeddings and manage chunking separately,
AntFly generates embeddings, summaries, chunks, and graph edges asynchronously at write
time. This makes the system AI-native — you write JSON documents, and the system handles
the vector/search plumbing.

### Leader-Only Background Work

Background tasks (enrichment, TTL cleanup) use a `LeaderFactory` pattern: only the
Raft leader runs them, tracked by an `atomic.Bool`. This avoids duplicate work across
replicas while keeping the background job lifecycle tied to leader election.

### Coverage Accounting, Not Doc Counts

Derived indexes track coverage via explicit per-source-unit state machines (pending →
in_flight → produced | skipped | terminal_failed), not raw `doc_count >= table_doc_count`.
This means the system knows exactly which documents have embeddings and which failed,
distinguishing "all work done" from "all desired outputs exist."

### Elastic License 2.0 Core, Apache 2.0 Everything Else

The core server is ELv2 (you can't offer AntFly as a managed service). SDKs, React
components, inference runtime, pgaf, docsaf, and evalaf are Apache 2.0 — keeping the
ecosystem permissive.

### Go 1.26 with GOEXPERIMENT=simd

Uses Go's hardware SIMD acceleration experiment for vector operations, wrapping
`go-highway` for x86 and ARM intrinsics. The `Makefile` bakes `GOEXPERIMENT=simd` into
all build targets.

## Comparison with Related Systems

- **vs Elasticsearch**: AntFly is Raft-based (not gossip-based), embeds ML inference
  (not plugins), and uses multi-Raft sharding (not primary/replica with split-brain
  risk). However, it's much younger and less battle-tested at scale.
- **vs Qdrant/Weaviate**: AntFly targets multimodal hybrid search with full-text +
  vector + graph in one system, not just vector search. It adds transactions, TTL,
  and RAG agents.
- **vs PostgreSQL + pgvector**: AntFly is purpose-built for AI workloads, not
  retrofitted. The pgaf extension brings AntFly search INTO Postgres, acknowledging
  that many users want to keep their data in Postgres.
- **vs Meerkat (Cloudflare's QuePaxa)**: Both use non-traditional consensus, but
  AntFly sticks with Raft rather than QuePaxa. The trade is known correctness vs.
  throughput.

## Companion Tools

- **docsaf**: Ingests content from filesystem, web crawl, git repos, and S3 into
  AntFly collections
- **evalaf**: LLM/RAG/agent evaluation framework ("promptfoo for Go")
- **pgaf**: PostgreSQL extension exposing AntFly search via `@@@` operator
- **memoryaf**: A2A/MCP-compatible memory service
- **Antfarm**: Web dashboard with playgrounds for search, RAG, knowledge graphs

## Sources

- Repository: https://github.com/antflydb/antfly
- Architecture docs: `docs/architecture.mdx`, `zig/DB.md`, `zig/HBC.md`, `zig/ALGEBRAIC.md`
- TLA+ specs: `specs/tla/`
- Raft implementation: `go/pkg/antfly/src/raft/`
- Zig data plane: `zig/pkg/antfly/src/`
- Inference runtime: `zig/pkg/inference/`
- C API boundary: `zig/CAPI.md`
