---
url: https://github.com/jeffhajewski/latticedb
title: LatticeDB
author: Jeff Hajewski
date_fetched: 2026-09-04
version: 0.15.0
topics:
  - databases-and-data
---

# LatticeDB

An embedded, single-file property-graph database written in Zig (~66K lines, zero runtime dependencies, MIT). It combines three search modes — graph traversal, HNSW vector similarity, and BM25 full-text search — in one engine and one Cypher-based query layer, so an application can query the same dataset by relationship, semantics, and text without running a server.

LatticeDB is built entirely on fixed-size 4 KB pages: a B+Tree for ordered storage, a write-ahead log with ARIES-style crash recovery, a buffer pool with per-page latches, and a page manager with checksums. The graph is not a custom storage engine but several B+Trees decomposed for access patterns — interned symbols, nodes, edges (a traversal tree plus a stable edge-ID index), a label index, and equality property indexes. Vector search uses a persistent HNSW index with heuristic neighbor selection, SIMD distance functions, and connection-page packing; full-text search is a BM25-scored inverted index with varint-encoded posting lists, skip pointers, 11-language stop words, and an English Porter stemmer.

Its most distinctive feature is a durable event layer inside the database: named streams and a built-in graph changefeed (`__lattice_changes`) that share the same WAL/transaction path as graph writes, so a committed write and its events become visible atomically. Bindings wrap a stable C ABI in Python, TypeScript, Go, and Java. The project targets relationship-heavy single-machine workloads — Graph RAG, agent memory, and local knowledge tools — and is explicit that it is a single-writer embedded engine, not a distributed database.

*Related: [[AntFly]] is the distributed counterpart of the same hybrid-search idea; [[The Log — Unifying Abstraction for Real-Time Data]] describes the log substrate LatticeDB internalizes.*
