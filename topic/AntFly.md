# AntFly

A distributed search engine and AI-native database that combines Raft consensus, hybrid
search (BM25 + vector + graph), and built-in ML inference into a single system. Documents
get embeddings, chunks, summaries, and graph edges automatically at write time — the
database handles the AI plumbing.

---

## Architecture

AntFly is a **multi-Raft** system with a Go/Zig dual-stack architecture. The Go layer
(`go/pkg/antfly/`) owns distributed coordination — Raft consensus, the HTTP API, metadata
management, and cluster orchestration. The Zig layer (`zig/pkg/antfly/src/`) owns the
local data plane on each shard — the storage engine (LMDB or a pure-Zig LSM tree), all
index types, query execution, and the inference runtime. They communicate via a stable C
ABI with typed binary codecs for hot paths (dense kNN, simple full-text search) to avoid
JSON serialization overhead.

**Consensus groups**: Separate Raft groups for metadata (cluster topology, schemas) and
storage (one per shard). The transport layer uses QUIC/HTTP3 for reduced connection
latency and head-of-line blocking avoidance.

**Index registry**: A plugin architecture where all indexes implement the `Index`
interface. Index types include `full_text` (BM25 via bleve), `embeddings` (HBC trees
with RaBitQ quantization), `graph` (relationship traversal), `remote` (proxy to other
indexes), and `algebraic` (a sparse-token sidecar for incremental aggregation).

**Write path**: Documents arrive via HTTP → metadata routes to owning shard → Raft leader
proposes to consensus group → committed to Pebble/LMDB → derived work (embeddings,
chunks, graph edges) appended to a sequence-based derived log → per-index workers advance
watermarks from that log.

## Key Techniques

### HBC Dense Vector Indexing

Hierarchical Balanced Clustering trees with several innovations over typical ANN indexes:
- **Write routing uses exact centroid distance**, not quantized error bounds (those are
  search-only machinery). This avoids unnecessary competitive selection on the insert path.
- **Bulk build** constructs final nodes once, quantizes once per finished node, and
  persists before tree construction — rather than per-vector online insertion.
- Multiple bulk-build strategies coexist (recursive, Hilbert-seeded, doc-key-seeded) and
  are measured against each other rather than chosen from assumptions.

### Algebraic Sparse-Token Sidecar

A research prototype that models the database as a sparse formal vector space.
Each document fact is tokenized into basis coordinates; queries compile into algebraic
operators; aggregates declare their algebraic structure (groups for COUNT/SUM, semilattices
for MIN/MAX, semirings for joins). This unifies relational query, materialized views,
incremental updates, indexes, and aggregation under one execution model. The prototype
is opt-in, planner-gated, and maintained as a sidecar index — base documents remain
canonical, and algebraic state is derived and rebuildable.

### Shard Splits Without Full Rebuild

Splitting a shard copies immutable state rather than replaying documents. Child docstores
are built page-for-page from LMDB source subtrees. Text indexes hand off complete segments
and rebuild only boundary-crossing mixed segments. Dense indexes classify HBC subtrees by
key range and hand off right-only subtrees unchanged. Sparse indexes copy right-only
posting blocks raw. Graph rebuilds from outgoing edge keys. Only the mixed boundary is
rebuilt — the dominant cost for a shard split is proportional to the boundary, not the
total shard size.

### Document Identity with Separator-Safe Encoding

Uses an order-preserving binary escape codec so arbitrary byte sequences (including NUL,
0xFF, and delimiter-shaped bytes) can serve as document IDs without corrupting internal
key structure. Layered identity: `canonical_doc_id` (deterministic per shard) + `doc_ordinal`
(compact u32 for Roaring bitmap efficiency). All index families converge on `doc_ordinal`
as the shared internal identity for cross-index filtering, replacing string document-ID
lists with dense bitmaps.

### Deterministic Simulation + TLA+ Verification

A full in-memory deterministic simulation harness (`go/pkg/antfly/src/sim/`) with
configurable network faults, mock clocks, and Jepsen-inspired chaos scenarios. Four
distributed protocols are formally specified and model-checked with TLA+: distributed
transactions, optimistic concurrency with 2PC, Raft snapshot transfer, and shard split
coordination. This dual approach — simulation for complex scenario testing, TLA+ for
protocol-level correctness — is unusually rigorous for an early-stage database.

## Design Decisions

**Write-time enrichment over query-time**: Embeddings, chunks, summaries, and graph
edges are generated asynchronously at write time rather than left to the user or
computed at query time. This makes the system AI-native but requires careful coverage
accounting to distinguish "all work done" from "all desired outputs exist."

**Leader-only background work**: Background tasks use a `LeaderFactory` pattern tied to
Raft leadership, tracked by `atomic.Bool`. This avoids duplicate enrichment work across
replicas while keeping lifecycle tied to leader election.

**Go for distribution, Zig for the hot path**: Rather than building everything in one
language, the system splits responsibility: Go handles the inherently distributed parts
(Raft, cluster management, HTTP API) while Zig owns the performance-critical local data
plane. The C ABI boundary uses typed binary codecs for hot paths (dense kNN, simple
full-text) to minimize serialization overhead. This dual-language approach is unusual
and introduces complexity at the boundary, but lets each language do what it's best at.

**ELv2 core, Apache 2.0 ecosystem**: The core server is Elastic License 2.0 (can't offer
AntFly as a managed service). All SDKs, React components, the inference runtime, pgaf,
docsaf, and evalaf are Apache 2.0 — keeping the developer ecosystem permissive.

**SIMD acceleration via Go experiment**: Uses `GOEXPERIMENT=simd` for hardware vector
operations on x86 and ARM, wrapping `go-highway`. Baked into every build target in the
Makefile.

## Comparison Notes

Unlike [[Recoll]] or [[QMD]] which are desktop/local search tools, AntFly is a
distributed system designed for production AI workloads at scale. Unlike [[Streambed]]
(a Postgres-to-Iceberg CDC pipeline), AntFly is an operational database, not an
analytics connector. Its multi-Raft design contrasts with [[Meerkat — QuePaxa Consensus
at Cloudflare]], which uses leader-optional QuePaxa for throughput over Raft's stronger
consistency guarantees. Like [[Nubase]], it bundles backend infrastructure (database,
auth, storage) for AI-native applications.

The algebraic sidecar research is conceptually related to [[Context Graphs]]' thesis that
typed edges beat vector similarity for retrieval — both treat database state as structured
symbolic entities rather than opaque embeddings.

#tool #project #database #search #distributed-systems #agents

---
*Sources: [[raw/antfly]]*
*Last updated: 2026-07-11*
