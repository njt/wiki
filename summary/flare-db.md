---
url: https://github.com/flare-db/flare-db
title: "FlareDB"
author: Ganesh Sivakumar
date_fetched: 2026-07-08
date_published: 2025
---

FlareDB is an Apache Beam-native streaming database written in Rust (v0.1.8,
Apache 2.0). It accepts Beam pipeline jobs submitted via a Java SDK runner,
fuses the pipeline graph into executable stages, and executes them — either
natively for core transforms like Impulse and GroupByKey, or by delegating to a
Java Beam SDK harness.

PCollections are stored as typed Arrow RecordBatches in an embedded LSM-tree
database (Tonbo), with schemas auto-derived from element types. This
PCollection-as-database model means intermediate results persist and are
queryable, unlike most Beam runners that hold them as opaque in-memory blobs.

The fusion algorithm (GreedyPipelineFuser) groups SDK transforms into executable
stages greedily, treating runner-side transforms (GroupByKey, Impulse) as
materialization boundaries. A deduplication pass injects synthetic Flatten
transforms for multi-producer PCollections — a correctness detail often missed
in naive fusion implementations.

v0.1.0 is single-node, single-port, bounded-sources-only on Global Window. ~8
gRPC endpoints are stubbed, state/timer APIs are not implemented, and the
executor has a known correctness bug around failure-after-execution-mark.
Distribution, watermarks, and event-time processing are roadmap items.

The project is a solo effort: roughly 4,000 lines of hand-written Rust for the
core engine, plus a small CLI and a Java runner SDK.
