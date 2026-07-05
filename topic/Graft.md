# Graft

SQLite replicated to the edge -- partially, as needed. A stateless system built on object storage that integrates directly into SQLite's commit path via a custom VFS, enabling distributed writes without requiring a cluster database.

---

## Key Quotes

> "A stateless system built on top of object storage, designed to replicate data to and from the edge."

> "Graft's Splinter-based changesets are self-contained, easily distributable"

## Key Themes

#database #git-for-data

Graft occupies an interesting position in the replicated database landscape. Its comparison page (which the annotation flags as "a juicy source") maps the field clearly:

- **vs mvSQLite**: Both use page-level versioning, but mvSQLite needs FoundationDB while Graft uses object storage
- **vs Litestream**: Litestream backs up WAL frames; Graft integrates into the commit path for distributed writes
- **vs cr-sqlite**: CRDTs give automatic conflict resolution; Graft is schema-agnostic but pushes conflict handling to the application
- **vs Turso/libSQL**: Log replay vs. partial replication
- **vs rqlite**: Raft consensus across stateful nodes vs. stateless on object storage

The "stateless on object storage" architecture is the key differentiator. It means Graft doesn't need a running cluster -- it uses S3 (or equivalent) as the coordination point. This is architecturally simpler and operationally cheaper than anything requiring consensus nodes.

Connects to [[Dolt]] (different philosophy -- full Git-for-data vs. edge replication) and the broader question of how AI agents running at the edge should manage local state.

## Critical Analysis

The partial replication story is compelling -- you don't always need the full database at every edge node, and fetching pages lazily is the right approach for many mobile and IoT workloads. The schema-agnostic design is both a strength (works with any SQLite schema) and a weakness (no automatic conflict resolution like cr-sqlite's CRDTs). The "application handles conflicts" approach is honest but puts significant burden on developers. Multi-language support (Python, JS, Ruby, Swift) suggests serious ambitions for the edge/mobile space. The comparison page alone is worth reading for anyone evaluating local-first database options.

---
*Sources: [[summary/graft]]*
*Last updated: 2026-05-14*
