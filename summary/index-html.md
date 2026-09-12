---
url: https://semiceu.github.io/LinkedDataEventStreams/releases/1.0.0/index.html
title: Linked Data Event Streams (LDES) — Consumer Specification v1.0
author: SEMIC (Semantic Interoperability Community)
date_fetched: 2026-08-06
topics:
  - databases-and-data
---

# Linked Data Event Streams — Specification Summary

The LDES consumer specification defines how clients replicate and synchronize with an append-only stream of RDF data published as one or more HTTP resources. It is part of a broader initiative by SEMIC to balance rich queryable APIs against static data dumps by making the event stream itself the foundational API.

## Core Model

An **LDES** (`ldes:EventStream`) is a collection of immutable **members** (each a set of RDF quads) published through one or more HTTP **nodes** (`tree:Node`) organized as a **search tree**. The specification picks up TREE hypermedia concepts from W3C: a root node provides context (SHACL shape, retention policies, timestamp/sequence paths), while root and subsequent nodes carry members and relations to other nodes.

## Synchronization Algorithm

A client takes an IRI and enters an initialization run, dereferencing the IRI to discover the root node and build initial state. Subsequent runs consult state and traverse the tree. The spec mandates two modes (unordered and ordered ascending), with ordered mode requiring `ldes:timestampPath` and/or `ldes:sequencePath`.

State management ensures each member is emitted exactly once. Immutable nodes (detected via `ldes:immutable`, `Cache-Control: immutable`, or time-bounded relations) are fetched at most once. Clients MUST support at least N-Quads, N-Triples, TriG, Turtle, and JSON-LD formats, implement retry with back-off for 4xx/5xx status codes, and process `410 Gone` as empty.

## Context Information

Context extracted from the root node includes: chronological ordering (`ldes:timestampPath`, `ldes:sequencePath`), the member SHACL shape (`tree:shape`), version semantics with create/update/delete objects, transaction grouping with finalization flags, and retention policies describing which subset of history a view retains. Retention policies use duration-based windows (`ldes:fullLogDuration`, `ldes:versionDuration`, `ldes:versionDeleteDuration`) and version counts (`ldes:versionAmount`).

## Key Design Decisions

- **Append-only immutability**: Members cannot be updated or removed once published, making the stream a durable log.
- **Polling over push**: The `ldes:pollingInterval` property governs synchronization cadence; the architecture is pull-based at heart.
- **Version order is separate from chronological order**: Versions can be published out of order (`ldes:versionTimestampPath`, `ldes:versionSequencePath`), acknowledging that publication time ≠ semantic time.
- **The search tree as traversal structure**: TREE relations (`GreaterThan`, `LessThan`, etc.) partition the event stream by time, enabling clients to efficiently seek to relevant segments.
