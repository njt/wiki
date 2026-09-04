# LatticeDB

LatticeDB is an embedded, single-file property-graph database that puts graph traversal, HNSW vector search, and BM25 full-text search behind one Cypher-based query layer. Written in Zig (~66K lines, zero runtime dependencies, MIT, v0.15.0), it is built for relationship-heavy single-machine workloads — Graph RAG, agent memory, and local knowledge tools — and is unusually honest that it is a single-writer embedded engine, not a distributed database. Its most interesting idea is that an event log (durable streams plus a graph changefeed) is a first-class database primitive sharing the same WAL path as graph writes, not a bolted-on consumer.

---

## Architecture

LatticeDB is a classic layered storage engine in the [[ARIES — Write-Ahead Logging Recovery]] tradition, rebuilt around one uniform primitive: **everything is a 4 KB page**. B+Tree nodes, WAL frames, the file header, and freelist entries are all pages, so there is one caching layer, one I/O path, and one checksum format (`docs/00_introduction.md`).

The stack, bottom-up (`src/storage/`): a VFS abstraction (`vfs.zig`, with an in-memory `memory_vfs.zig`), a page manager with checksums and freelist (`page_manager.zig`), a buffer pool with per-page reader-writer latches and a clock eviction policy (`buffer_pool.zig`), a B+Tree (`btree.zig`, 3.1K lines) with explicit overflow pages for large values, a WAL (`wal.zig`), a checkpointer, and an ARIES-style recovery manager (`recovery.zig`). Above it sits a transaction manager (`transaction/manager.zig`) that writes records with `prev_lsn` back-pointers for undo chaining, plus MVCC machinery (`transaction/mvcc.zig`).

The graph is **not** a bespoke storage format — it is several B+Trees decomposed by access pattern (`docs/09_graph_storage.md`): a two-way symbol table for string interning, a node store keyed by `u64` ID, an edge store that writes *two* traversal keys (`(source, dir, type, target, edge_id)` outgoing and `(target, dir, type, source, edge_id)` incoming, big-endian so range scans are contiguous) plus a *separate* edge-ID index holding the payload once, a label index, and equality property indexes. Stable edge IDs give parallel edges first-class identity. The query engine is a Volcano iterator model (`src/query/operators/`, `docs/12_query_execution.md`): lexer → parser → semantic analyzer → planner → a pull-based operator tree over fixed 16-slot rows allocated from an arena.

## Key Techniques

- **One query layer over three index types.** `<=>` (vector distance) and `@@` (full-text match) are first-class operators the planner recognizes and rewrites into specialized HNSW/FTS operators rather than a scan-and-filter fallback. A single query can traverse `(chunk)-[:PART_OF]->(doc)-[:AUTHORED_BY]->(author)` while also filtering on `chunk.embedding <=> $q < 0.3` and `doc.content @@ "neural networks"` — the README's headline example.
- **Byte-balanced B+Tree splits.** Leaf splits choose the split point by serialized byte size, not entry count, so both halves stay valid when a leaf mixes small keys with large values (`docs/04_btree.md`).
- **Persistent HNSW with connection-page packing.** Connections are stored in shared "connection pool" pages (`src/vector/hnsw.zig`, `HnswNodeEntry` serialized to 21 bytes, `ConnectionPoolPageHeader`), giving ~4.5× memory reduction, with heuristic neighbor selection (HNSW Algorithm 4) and pre-normalized dot-product cosine. Distances use Zig's `@Vector` SIMD (`src/vector/distance.zig`, 8-wide for AVX-256). Note a doc/code drift: `docs/10_vector_search.md` still lists "no incremental persistence, full rebuild on restart" as a limitation, but the code clearly persists the graph to pages.
- **Opt-in adjacency cache.** `src/graph/adjacency_cache.zig` maps `NodeId → []CachedEdge` in memory, bypassing the B+Tree on repeated traversals (~50 ns vs ~32 µs), invalidated on edge mutation. This is what produces the headline 23×–2,819× speedups over SQLite recursive CTEs at depth.
- **The changefeed is semantic, not page-level.** `__lattice_changes` is populated from transaction diffs (`node.insert`, `edge.property_set`, …), and oversized property values degrade to summary maps rather than blowing the WAL record limit (`docs/14_durable_streams.md`).
- **An engine conformance spec** (`docs/13_engine_conformance.md`). It freezes observable behavior — stable edge IDs, missing-vs-NULL, single-writer contention — in a black-box conformance suite so a future Go engine can be swapped in without matching the on-disk format. Rare discipline for a solo database.

## Design Decisions

- **Embedded single-writer over distribution.** One process owns the file; a second writer on the same handle fails with a contention error. Continuous backup and restore exist, but "that is backup rather than clustering." The README's "When to Use Something Else" section is a candid admission that this buys speed and zero-config at the cost of multi-client and multi-machine scale.
- **Zero-copy pages over object models.** Nodes and B+Tree entries are read and written directly at byte offsets in page buffers, with `extern struct` headers carrying compile-time `@sizeOf` assertions. The "node" is a lens over page bytes.
- **Honest about isolation.** The internal transaction manager has snapshot-oriented MVCC, but the conformance doc explicitly downgrades the public guarantee to "one live read-write transaction per handle," warning callers not to depend on stronger un-exposed behavior. Most projects paper over this gap; LatticeDB documents it.
- **Deliberate scope cuts.** `OPTIONAL MATCH` and `CALL` are unimplemented; stemming is English-only. The `hash_embed` helper is a deterministic placeholder, and the README warns it is *not* semantic (similar text does not produce nearby vectors) — an unusual refusal to oversell.
- **Simplicity over maximum concurrency.** Per-page latches with latch crabbing, a mutex-guarded transaction manager held for microseconds — no lock hierarchies, no lock-free structures.

## Comparison Notes

[[AntFly]] is the same hybrid graph+vector+BM25 thesis executed at the opposite pole: a distributed multi-Raft system with write-time AI enrichment and Go/Zig split, where LatticeDB is an embedded single-file library with no consensus, no cluster, and no generated embeddings. They share the Zig hot path and the "one engine, three search modes" vision.

[[Just Brute Force Your Embeddings]] argues most teams don't need an ANN index; LatticeDB is the rung *above* brute force — its claimed 0.83 ms at 1M vectors with 100% recall is competitive with FAISS and Weaviate — but it only matters once you also want graph traversal and BM25 in the same query, not just vectors. [[zvec]] is the pure embedded-vector answer (billions of vectors, no graph); [[NornicDB]] adds vectors to a Neo4j-compatible graph but stays server-shaped.

[[The Log — Unifying Abstraction for Real-Time Data]] is the theory LatticeDB internalizes: durable streams and the graph changefeed make the database itself the event log, with committed writes and their events atomic in one WAL. It's the same tables/events duality Kreps describes, minus the distributed transport.

On the query language, LatticeDB's Cypher subset and `[[Icebug Format]]`'s Cypher schema both treat Cypher as the graph lingua franca, but LatticeDB extends it with `<=>` and `@@` operators rather than staying pure-Cypher.

#tool #project #database #graph #vector #search #memory #agents

---
*Sources: [[raw/latticedb]], [[summary/latticedb]]*
*Last updated: 2026-09-04*
