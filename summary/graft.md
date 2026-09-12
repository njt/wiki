---
title: "Graft"
url: https://graft.rs/
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - databases-and-data
---

# Graft: SQLite Replication at the Edge

## Summary
Graft is an open-source transactional storage engine designed for efficient data synchronization at the edge. It replicates SQLite databases partially, as needed, to clients.

## Core Architecture
Graft integrates directly into SQLite's commit path via a custom VFS. It uses page-level versioning for lazy fetching and partial views, built on object storage rather than requiring a cluster database.

## Key Comparisons

**vs mvSQLite:** Both use page-level versioning. mvSQLite depends on FoundationDB; Graft uses object storage. "Graft's Splinter-based changesets are self-contained, easily distributable."

**vs Litestream:** Litestream focuses on WAL frame backup. Graft integrates into SQLite's commit enabling distributed writes.

**vs cr-sqlite:** cr-sqlite uses CRDTs for automatic conflict resolution. Graft is schema-agnostic but requires application-level conflict handling.

**vs Cloudflare D1:** D1 is centralized managed service via HTTP. Graft enables decentralized, embedded edge replication.

**vs Turso/libSQL:** Checkpoint and log replay replication. Graft supports partial replication and schema-agnostic structures.

**vs rqlite:** Raft consensus across stateful nodes. Graft is stateless on object storage for edge distribution.

## Core Distinction
"A stateless system built on top of object storage, designed to replicate data to and from the edge."

## Language Support
Python, JavaScript, Ruby, Swift, and direct manual implementation.
