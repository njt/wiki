---
url: https://github.com/antflydb/antfly
title: "AntFly"
author: AJ Roetker and contributors
date_fetched: 2026-07-11
date_published: 2025
---

AntFly is a distributed search engine and AI-native database built on etcd's Raft
consensus library. It combines full-text search (BM25), vector similarity (HBC trees
with RaBitQ quantization), and graph traversal over multimodal data — text, images,
audio, and video — in a single system.

The architecture uses a **multi-Raft** design: separate consensus groups for metadata
and for each shard, with QUIC (HTTP/3) as the transport layer. The system is
multi-language: **Go** owns distribution (Raft, HTTP API, cluster orchestration);
**Zig** owns the local data plane (storage engine, all index types, query execution,
inference runtime); they communicate via a stable C ABI. Additional SDKs exist in
TypeScript, Rust, and Python.

AntFly is optimized for **write-time enrichment**: embeddings, summaries, chunks, and
graph edges are generated automatically when documents are written, rather than
requiring users to pre-compute them. Background work runs only on the Raft leader via
a `LeaderFactory` pattern. Index coverage uses explicit per-document state machines
rather than raw document counts.

The **index system** is plugin-based: full-text (bleve), vector embeddings
(RaBitQ-quantized HBC trees), graph traversal, remote proxy, and an algebraic
sparse-token sidecar. A research thread models the database as a sparse formal vector
space, with query plans as algebraic operator compositions and aggregates as folds
into monoids, groups, or semirings.

AntFly includes **TLA+ formal specifications** for four protocols (transactions, OCC
2PC, snapshot transfer, shard split) and a **deterministic simulation harness** in Go
for Jepsen-style chaos testing. The shard-split design minimizes data movement with
page-level copies, segment handoffs, and subtree transfers rather than full rebuilds.

The core server is **Elastic License 2.0**; SDKs, React components, the inference
runtime, and companion tools (docsaf, evalaf, pgaf, memoryaf, Antfarm dashboard) are
**Apache 2.0**. The pgaf PostgreSQL extension exposes AntFly search via a `@@@`
operator, letting users keep data in Postgres while using AntFly's search capabilities.
