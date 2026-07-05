---
url: https://github.com/orneryd/NornicDB
date_fetched: 2026-07-05
backfilled: true
---

**Graph, vector, and historical truth in one database**

  Neo4j-compatible • Hybrid graph + vector retrieval • Historical reads via MVCC

  *Achieving Psygnosis for AI*
  

Multi-arch support: CPU | CUDA | Metal | Vulkan


Quick Start • What It Is • Why NornicDB • Benchmarks • Features • Docs • Comparison • Contributors

```
# Homebrew
brew tap --trust orneryd/nornicdb && brew install nornicdb && brew services start nornicdb
# arm64 / Apple Silicon
docker run -d --name nornicdb -p 7474:7474 -p 7687:7687 -v nornicdb-data:/data timothyswt/nornicdb-arm64-metal-bge:latest
# amd64 / CPU only
docker run -d --name nornicdb -p 7474:7474 -p 7687:7687 -v nornicdb-data:/data timothyswt/nornicdb-amd64-cpu-bge:latest
```
Open http://localhost:7474 for the admin UI. For NVIDIA CUDA hosts, use `timothyswt/nornicdb-amd64-cuda-bge:latest`. For Vulkan hosts, use `timothyswt/nornicdb-amd64-vulkan-bge:latest`.

Note: Docker on macOS does not expose Metal acceleration. The Apple Silicon image still runs, but GPU acceleration on macOS requires a native install from the releases page or a local build.



Writing queries?Start with the Hot-Path Cypher Cookbook — proven query shapes that route through the executor's specialized fast paths.🤖

Building with Claude / agents?The`docs/skills/`directory contains agent-ready skill files for every Cypher surface: query shapes, decay/promotion policies, managed embeddings, vector & hybrid search, and RAG procedures. Drop them into`.claude/skills/`to make agents fluent in NornicDB.

NornicDB is a graph database for workloads that need graph traversal, vector retrieval, and historical truth in the same system. It speaks Neo4j's language through Bolt and Cypher, exposes REST, GraphQL, and gRPC interfaces, and can preserve Qdrant-style client workflows where that helps migration.

The architecture draws from research in Temporal GraphRAG, agent memory systems, event-sourced decision memory, and persistence semantics for AI agents. These ideas appear in NornicDB as graph-native support for temporal facts, canonical knowledge versioning, replayable history, policy-driven memory retention, and audit-friendly retrieval.

It is built for knowledge systems, agent memory, Graph-RAG, and canonical truth stores where semantic search is only part of the query. The design goal is not to bolt a vector store onto a graph database. The design goal is one execution path for graph, vector, temporal, and audit-oriented workloads.

- **Neo4j-compatible by default**: Bolt + Cypher support for existing drivers and applications.
- **Built for AI-native workloads**: vector search, memory decay, and auto-relationships are first-class features.
- **Graph, vector, and ledger semantics in one engine**: hybrid retrieval, graph traversal, canonical graph ledger modeling, tritemporal facts, as-of reads, txlog queries, and receipts do not require a second database.
- **Protocol flexibility without splitting the system**: REST, GraphQL, Bolt/Cypher, Qdrant-compatible gRPC, and additive Nornic gRPC live on the same platform.
- **Hardware-accelerated execution**: Metal/CUDA/Vulkan pathways for high-throughput graph + semantic workloads.
- **Operational flexibility**: full images (models included), BYOM images, and headless API-only deployments.

NornicDB is being used in internal production deployments for stack-consolidation workloads where graph traversal, vector retrieval, and auditability need to live in the same system.

- **Agent and Graph-RAG systems**: replacing a Neo4j + Qdrant + embeddings stack with a single deployment for task tracking, dependency graphs, and retrieval pipelines.
- **Translation and evaluation workflows**: replacing a document store plus embeddings pipeline with a single deployment for graph-native retrieval and faster aggregation paths.

NornicDB implements Snapshot Isolation at the storage layer. Each transaction is anchored to a specific MVCC version, so point reads, label scans, and snapshot-visible graph traversals resolve against the same committed view of the graph.

- **Repeatable reads within a transaction**: transactions see their own buffered writes, but not commits that land after their read snapshot.
- **Conflict detection at commit**: concurrent graph mutations against the same logical state fail with a normalized- `ErrConflict`instead of silently overwriting newer data.
- **Explicit historical reads**: MVCC pruning preserves the current head and a retained floor per logical key; requests below that retained floor fail safely with- `ErrNotFound`.
- **Search remains current-state focused**: current search paths are intentionally separate from historical MVCC state.

See transaction implementation details, historical reads and MVCC retention, and the canonical graph ledger guide.

**LDBC Social Network Benchmark** (M3 Max, 64GB):

| Query Type | NornicDB | Neo4j | Speedup | 
|---|---|---|---|
| Message content lookup | 6,389 ops/sec | 518 ops/sec | 12x | 
| Recent messages (friends) | 2,769 ops/sec | 108 ops/sec | 25x | 
| Avg friends per city | 4,713 ops/sec | 91 ops/sec | 52x | 
| Tag co-occurrence | 2,076 ops/sec | 65 ops/sec | 32x | 

See full benchmark results for complete methodology and additional workloads.


Hybrid retrieval is where NornicDB is materially different from vector-only stacks: the query shape is vector search followed by graph expansion in the same engine.

**Local benchmark** (67,280 nodes, 40,921 edges, 67,298 embeddings, HNSW CPU-only index):

| Workload | Transport | Throughput | Mean | P50 | P95 | P99 | Max | 
|---|---|---|---|---|---|---|---|
| Vector only | HTTP | 19,342 req/s | 511 us | 470 us | 750 us | 869 us | 1.02 ms | 
| Vector only | Bolt | 22,309 req/s | 444 us | 428 us | 629 us | 814 us | 968 us | 
| Vector + 1 hop | HTTP | 11,523 req/s | 859 us | 699 us | 1.54 ms | 3.46 ms | 4.71 ms | 
| Vector + 1 hop | Bolt | 13,291 req/s | 747 us | 637 us | 1.29 ms | 3.24 ms | 4.47 ms | 

**Remote benchmark** (GCP, 8 vCPU, 32 GB RAM):

- Vector only: ~110.7 ms P50
- Vector + 1 hop: ~112.9 ms P50
- The delta between local and remote matched network RTT closely enough that end-to-end latency was network-bound rather than compute-bound.

This point is: once vector search plus one-hop traversal stays in low single-digit milliseconds locally, the bottleneck shifts from retrieval logic to deployment topology.

See the hybrid retrieval benchmark write-up for methodology, caveats, and reproduction queries, and see Graph-RAG: NornicDB vs Typical for the architectural implications.

NornicDB is currently being utilized by researchers at **UCLouvain** to map large-scale **Cyber-Physical Systems** (CPS).

In benchmarks performing **Automata Learning (L*)**—a high-iteration logic process where an LLM acts as a "Deterministic Teacher" or Oracle—NornicDB outperformed industry-standard graph databases by a significant margin:

- **Efficiency:**- **2.2x Faster**than Neo4j in total execution time for formal logic mapping.
- **Throughput:**Successfully handled- **1,443 state-transition queries**in ~32 seconds (Avg 22.69ms per full reasoning loop).

| DATABASE | CALLS | AVG TIME (ms) | TOTAL (s) | 
|---|---|---|---|
| NornicDB | 1443 | 22.69 | 32.74 | 
| Neo4j | 1443 | 50.20 | 72.43 | 

- **Hybrid execution model (streaming fast paths + general engine)**: NornicDB uses shape-specialized streaming executors for common traversal/aggregation patterns while retaining a general Cypher path for coverage and correctness.
- **Runtime parser mode switching**: the default- `nornic`parser is optimized for low-overhead hot-path routing, while- `antlr`mode prioritizes strict parsing and diagnostics when debugging and validation matter more than throughput.
- **Measured parser-path deltas on benchmark suites**: internal Northwind comparisons show large overhead differences on certain query shapes when full parse-tree paths are used, which is why the production default remains the custom parser path.
- **HNSW build acceleration from insertion-order optimization**: BM25-seeded insertion order reduced a 1M embedding build from ~27 minutes to ~10 minutes (~2.7x) in published tests by reducing traversal waste during construction, without changing core quality knobs.
- **Shared seed strategy across indexing stages**: the same lexical seed extraction supports HNSW insertion ordering and improves k-means centroid initialization spread for vector pipeline efficiency.

Read more:

```
# Apple Silicon (includes bge-m3 embedding model)
docker run -d --name nornicdb \
  -p 7474:7474 -p 7687:7687 \
  -v nornicdb-data:/data \
  timothyswt/nornicdb-arm64-metal-bge:latest  # Apple Silicon
  # timothyswt/nornicdb-amd64-cuda-bge:latest  # NVIDIA GPU
```
Open http://localhost:7474 for the admin UI.

Need a different image/profile (Heimdall, BYOM, CPU-only, Vulkan, headless)?

```
git clone https://github.com/orneryd/NornicDB.git
cd NornicDB
go build -o nornicdb ./cmd/nornicdb
./nornicdb serve
```
Use any Neo4j driver — Python, JavaScript, Go, Java, .NET:

```
from neo4j import GraphDatabase
driver = GraphDatabase.driver("bolt://localhost:7687")
with driver.session() as session:
    session.run("CREATE (n:Memory {content: 'Hello NornicDB'})")
```
- **12x-52x faster**on published LDBC workloads (same hardware comparisons).
- **Native graph + vector**in one engine (no separate vector sidecar required).
- **GPU acceleration paths**(Metal/CUDA/Vulkan) for semantic + graph workloads.
- **Drop-in compatibility**via Bolt + Cypher for existing applications.
- **Canonical graph ledger model**for temporal validity, tritemporal fact modeling, as-of reads, and audit-oriented mutation tracking.

🚚

Migrating?Neo4j → NornicDB skill · runnable Python/Go/Node scripts in`scripts/migration/neo4j/`.

- **Graph + vector in one engine**: combine semantic retrieval with native graph traversal and Cypher queries.
- **Qdrant gRPC compatibility preserved**: keep Qdrant-style gRPC workflows while adding graph-native capabilities.
- **Hybrid retrieval built in**: vector + BM25 fusion and optional reranking in the same query pipeline.
- **Canonical truth modeling**: versioned facts, temporal validity windows, tritemporal facts, and as-of reads for governance-heavy use cases.
- **Protocol flexibility**: use REST, GraphQL, Bolt/Cypher, Qdrant-compatible gRPC, and additive Nornic gRPC on one platform.

🚚

Migrating?Qdrant → NornicDB skill · gRPC surface skill · runnable Python/Go/Node scripts in`scripts/migration/qdrant/`.

Retention policy enforcement is available, but it is disabled by default and must be explicitly enabled. When retention is off, NornicDB does not create the retention manager and does not start the retention sweep background worker. When enabled, retention supports label-aware policy evaluation, legal holds, GDPR erasure tracking, and admin APIs.

See Retention Policies and Configuration.

Designed to work with existing Neo4j drivers and Bolt/Cypher workflows, with minimal or no application changes for supported query shapes.

- **Bolt Protocol**— Use official Neo4j drivers
- **Cypher Queries**— Full query language support
- **Schema Management**— Constraints, indexes, vector indexes
- **Qdrant gRPC API Compatible**— Works with Qdrant-style gRPC vector workflows

🤖

Agent skill:Bolt Client — connection defaults, retry classification, MERGE under concurrent writers, batch sizing.

Profile-driven decay and promotion scoring with the Ebbinghaus-Roynard four-layer decomposition. The engine does not hardcode cognitive tiers. Operators model their own labels and lifecycle rules using Cypher DDL.

Typical deployments map the four-layer decomposition onto labels such as:

- **Knowledge**: durable fact labels using- `NO DECAY`or neutral profiles
- **Memory**: episodic/session labels using bounded half-life decay
- **Wisdom**: stable directive labels using conservative decay plus promotion rules
- **Evidence/links**: edge types with their own decay and suppression behavior

Those categories are conventions, not built-in engine classes. NornicDB provides the authoring and diagnostics surface:

- `CREATE/ALTER/DROP/SHOW DECAY PROFILE`
- `CREATE/ALTER/DROP/SHOW PROMOTION PROFILE`
- `CREATE/ALTER/DROP/SHOW PROMOTION POLICY`
- `decayScore(entity)`,- `decay(entity)`,- `policy(entity)`,- `reveal(entity)`
- `CALL nornicdb.knowledgepolicy.info|profiles|policies|resolve|deindexStatus()`

```
CREATE DECAY PROFILE working_memory OPTIONS {
  halfLifeSeconds: 604800,
  function: 'exponential',
  visibilityThreshold: 0.10
}
CREATE DECAY PROFILE session_retention
FOR (n:SessionRecord)
APPLY {
  DECAY PROFILE 'working_memory'
  n.tenantId NO DECAY
}
MATCH (n:SessionRecord) WHERE decayScore(n) > 0.5
RETURN n ORDER BY decayScore(n) DESC
```
📖 Deep dive: Knowledge-Layer Policies, Decay Profiles, Promotion Policies, and Ebbinghaus-Roynard Bootstrap.

🤖

Agent skills:Knowledge Policies · Decay Tuning · Promotion Policies

NornicDB weaves connections automatically:

- **Embedding Similarity**— Related concepts link together
- **Co-access Patterns**— Frequently queried pairs connect
- **Temporal Proximity**— Same-session nodes associate
- **Transitive Inference**— A→B + B→C suggests A→C

Native semantic search with GPU acceleration and hybrid retrieval support.

📖 Deep dive: Vector Search Guide and Qdrant gRPC Endpoint.

🤖

Agent skills:Vector & Full-Text Search · Managed Embeddings · RAG Procedures

**Cypher (Neo4j-compatible):**

```
CALL db.index.vector.queryNodes('embeddings', 10, 'machine learning guide')
YIELD node, score
RETURN node.content, score
```
**Hybrid search (REST):**

```
curl -X POST http://localhost:7474/nornicdb/search \
  -H "Content-Type: application/json" \
  -d '{"query": "machine learning", "limit": 10}'
```
More API entry points:

- **GraphQL**hybrid search:- `POST /graphql`with- `search(query, options)`
- **gRPC**(Qdrant-compatible):- `Points.Search`/- `Points.Query(Document.text)`
- **Nornic native gRPC**:- `NornicSearch/SearchText`(additive client)
- See `docs/user-guides/nornic-search-grpc.md`for additive proto setup without forking Qdrant drivers.

Built-in AI that understands your database.

```
# Enable Heimdall
NORNICDB_HEIMDALL_ENABLED=true ./nornicdb serve
```
**Natural Language Queries:**

- "Get the database status"
- "Show me system metrics"
- "Run health check"

**Plugin System:**

- Create custom actions the AI can execute
- Lifecycle hooks (PrePrompt, PreExecute, PostExecute)
- Database event monitoring for autonomous actions
- Inline notifications with proper ordering

See Heimdall AI Assistant Guide and Plugin Development.

950+ built-in functions for text, math, collections, and more. Plus a plugin system for custom extensions.

```
// Text processing
RETURN apoc.text.camelCase('hello world')  // "helloWorld"
RETURN apoc.text.slugify('Hello World!')   // "hello-world"
// Machine learning
RETURN apoc.ml.sigmoid(0)                  // 0.5
RETURN apoc.ml.cosineSimilarity([1,0], [0,1])  // 0.0
// Collections
RETURN apoc.coll.sum([1, 2, 3, 4, 5])      // 15
```
Drop custom `.so` plugins into `/app/plugins/` for automatic loading. See the APOC Plugin Guide.

All images available at Docker Hub.

| Image | Size | Description | 
|---|---|---|
| `timothyswt/nornicdb-arm64-metal-bge-heimdall` | 1.1 GB | Full- Embeddings + AI Assistant | 
| `timothyswt/nornicdb-arm64-metal-bge` | 586 MB | Standard- With BGE-M3 embeddings | 
| `timothyswt/nornicdb-arm64-metal` | 148 MB | Minimal- Core database, BYOM | 
| `timothyswt/nornicdb-arm64-metal-headless` | 148 MB | Headless- API only, no UI | 

| Image | Size | Description | 
|---|---|---|
| `timothyswt/nornicdb-amd64-cuda-bge` | ~4.5 GB | GPU + Embeddings- CUDA + BGE-M3 | 
| `timothyswt/nornicdb-amd64-cuda` | ~3 GB | GPU- CUDA acceleration, BYOM | 
| `timothyswt/nornicdb-amd64-cuda-headless` | ~2.9 GB | GPU Headless- API only | 
| `timothyswt/nornicdb-amd64-cpu` | ~500 MB | CPU- No GPU required | 
| `timothyswt/nornicdb-amd64-cpu-headless` | ~500 MB | CPU Headless- API only | 

**BYOM** = Bring Your Own Model (mount at `/app/models`)

```
# With your own model
docker run -d -p 7474:7474 -p 7687:7687 \
  -v /path/to/models:/app/models \
  timothyswt/nornicdb-arm64-metal:latest
# Headless mode (API only, no web UI)
docker run -d -p 7474:7474 -p 7687:7687 \
  -v nornicdb-data:/data \
  timothyswt/nornicdb-arm64-metal-headless:latest
```
For embedded deployments, microservices, or API-only use cases, NornicDB supports headless mode which disables the web UI for a smaller binary and reduced attack surface.

**Runtime flag:**

`nornicdb serve --headless`**Environment variable:**

`NORNICDB_HEADLESS=true nornicdb serve`**Build without UI (smaller binary):**

```
# Native build
make build-headless
# Docker build
docker build --build-arg HEADLESS=true -f docker/Dockerfile.arm64-metal .
```
```
# nornicdb.yaml
server:
  bolt_port: 7687
  http_port: 7474
  host: localhost
database:
  data_dir: ./data
  async_writes_enabled: true
  async_flush_interval: 50ms
  async_max_node_cache_size: 50000
  async_max_edge_cache_size: 100000
embedding:
  enabled: true
  provider: local # or ollama, openai
  model: bge-m3.gguf
  url: ""
  dimensions: 1024
embedding_worker:
  chunk_size: 8192
  chunk_overlap: 50
memory:
  decay_enabled: true
  decay_interval: 3600
  auto_links_enabled: true
  auto_links_similarity_threshold: 0.82
```
- **AI Agent Memory**— Persistent, queryable memory for LLM agents
- **Knowledge Graphs**— Auto-organizing knowledge bases
- **RAG Systems**— Vector + graph retrieval in one database
- **Graph-RAG for LLM Inference**— Simplify retrieval pipelines by combining graph traversal, hybrid search, and provenance in one engine
- **Session Context**— Decaying conversation history
- **Research Tools**— Connect papers, notes, and insights
- **Canonical Truth Stores**— Versioned facts, temporal validity, and append-only mutation history in a graph model
- **Financial Systems**— Loan/risk state reconstruction with as-of reads and audit receipts
- **Compliance & RegTech**— KYC/AML state changes, policy/rule versioning, and non-overlapping validity enforcement
- **Audit Platforms**— Correlate graph mutations to WAL sequence ranges and receipt hashes
- **AI Governance & Lineage**— Track model assertions, overrides, and fact provenance over time

Start with the docs hub for role/task navigation, then use the issue index for symptom-first troubleshooting:

- Public documentation site: https://orneryd.github.io/NornicDB/
- Documentation Hub
- Issue Index

| Guide | Description | 
|---|---|
| Getting Started | Installation & quick start | 
| Docker Image Quick Reference | Full runtime image matrix | 
| Hot-Path Cypher Cookbook | Proven query shapes for fast latency | 
| Agent Skills | Claude/agent skill files for the Cypher surface | 
| API Reference | Cypher functions & procedures | 
| User Guides | Complete examples & patterns | 
| Performance | Benchmarks vs Neo4j | 
| Neo4j Migration | Compatibility & feature parity | 
| Migration Scripts | Runnable Neo4j and Qdrant → NornicDB migrations (Python, Go, Node) | 
| Architecture | System design & internals | 
| Docker Guide | Build & deployment | 
| Development | Contributing & development | 

Additional deep dives referenced above:

- Hybrid query benchmarks
- Canonical graph ledger
- Historical reads and MVCC retention
- Cypher parser modes

| Platform | Category | Query Language Support (and protocol) | Native Vector Search | Canonical Graph + Temporal Ledger Pattern | Queryable Mutation Log + Receipts | Embedded/Self-Hosted Focus | 
|---|---|---|---|---|---|---|
| NornicDB | Graph + Vector + Canonical Ledger | Cypher via Bolt; also HTTP/GraphQL and gRPC (Qdrant-compatible + NornicSearch) | Yes | Yes | Yes | Yes | 
| Neo4j | Graph DB | Cypher via Bolt/HTTP | Yes | Partial (manual modeling) | Partial (logs exist, not first-class receipts model) | Server-first | 
| Memgraph | Graph DB | openCypher via Bolt/HTTP | Partial/varies by setup | Partial (manual) | Partial (manual/integration) | Server-first | 
| TigerGraph | Graph analytics DB | GSQL via REST++/native endpoints | Partial/extension-driven | Partial (manual) | Partial (manual/integration) | Server-first | 
| Qdrant | Vector DB | Qdrant query/filter API via gRPC/REST | Yes | No (not graph-native) | No | Server-first | 
| Weaviate | Vector DB | GraphQL + REST APIs | Yes | Partial (knowledge graph features, not Cypher property graph) | No | Server-first | 
| Amazon QLDB | Ledger DB | PartiQL via AWS API/SDK | No | Partial (ledger + temporal history, not graph-native) | Yes (ledger-native) | Managed service | 

Snapshot is capability-oriented and high-level; exact behavior depends on edition/configuration and workload design.


```
# Basic build
make build
# Headless (no UI)
make build-headless
# With local LLM support
make build-localllm
```
```
# Download models for Heimdall builds (automatic if missing)
make download-models        # BGE-M3 + qwen3-0.6b (~750MB)
make check-models          # Verify models present
# ARM64 (Apple Silicon)
make build-arm64-metal                  # Base (BYOM)
make build-arm64-metal-bge              # With BGE embeddings
make build-arm64-metal-bge-heimdall     # With BGE + Heimdall AI
make build-arm64-metal-headless         # Headless (no UI)
# AMD64 CUDA (NVIDIA GPU)
make build-amd64-cuda                   # Base (BYOM)
make build-amd64-cuda-bge               # With BGE embeddings
make build-amd64-cuda-bge-heimdall      # With BGE + Heimdall AI
make build-amd64-cuda-headless          # Headless (no UI)
# AMD64 CPU-only
make build-amd64-cpu                    # Minimal
make build-amd64-cpu-headless           # Minimal headless
# Build all variants for your architecture
make build-all
# Deploy to registry
make deploy-all             # Build + push all variants
```
```
# Build for other platforms from macOS
make cross-linux-amd64     # Linux x86_64
make cross-linux-arm64     # Linux ARM64
make cross-rpi             # Raspberry Pi 4/5
make cross-windows         # Windows (CPU-only)
make cross-all             # All platforms
```
- Neo4j Bolt protocol
- Configurable RBAC and oAuth support
- Cypher query engine (52 functions)
- Memory decay system
- GPU acceleration (Metal, CUDA)
- Vector & full-text search
- Auto-relationship engine
- HNSW vector index
- Metadata/Property Indexing
- SIMD Implementation
- Clustering support
- Sharding (Composite DB + Remote Constituents)
- Data Explorer UI (Browser query editor, semantic search, node details)
- GDPR Compliance
-  per-DB Search Index Overrides for BM2 and HNSW as independently levers for deferred or skipped construction (`docs/plans/nornicdb-admin-import-plan.md`)
- [*] GPU-assisted HNSW construction - Metal: [f8f09fdc]

- Bulk Import Tool
- [*] GPU-assisted HNSW construction with CPU-serving persistence parity (`docs/plans/gpu-hnsw-construction-plan.md`) - Cuda/Vulkan TBD
-  Neo4j-compatible end-to-end streaming execution + wrapper driver/ORM (`docs/plans/neo4j-compatible-streaming-driver-and-server-plan.md`)
-  UI enhancement backlog (search/config/admin UX improvements) (`docs/plans/ui-enhancements.md`)

Special thanks to everyone who helps make NornicDB better. See CONTRIBUTORS.md for a list of community contributors.

MIT License — See LICENSE.md for details.

Patent rights are handled via a defensive non-assertion grant in PATENTS.md. This keeps the project open for broad use (including commercial use) while adding patent retaliation protection.

See NOTICES.md for third-party license information, including bundled AI models (BGE-M3, Qwen2.5) and dependencies.

  *Psygnosis is a play on words or portmanteau meaning “mind" + "knowledge” in greek*
