# Icebug Format

A standardized property-graph interchange format that bridges the relational and graph worlds: it converts node/edge tables from DuckDB into CSR (Compressed Sparse Row) adjacency backed by Parquet files, with a companion `schema.cypher` that lets a graph engine query the data in-place from object storage without ingesting it first. Comes in two flavors — disk (Parquet) and memory (Apache Arrow) — sharing the same CSR encoding.

---

## Architecture

Icebug is a **format converter**, not a database. It reads relational tables, applies a deterministic pipeline, and outputs a self-describing graph layout.

The pipeline (`icebug_format/cli.py:395-705`):
1. **Discover** `nodes_*` and `edges_*` tables from the source DuckDB database
2. **Copy and sort** node tables by their first column (the primary key)
3. **Create dense mappings** — assign each node a zero-based `csr_index` via `row_number() OVER (ORDER BY pk) - 1`, per node type
4. **Join** edge source/target columns to the mapping tables (inner join — dangling edges are dropped)
5. **Build CSR indptr** — cumulative degree per source node via a CTE that ranges over all node IDs, LEFT JOINed to degree counts, with cumulative sum window function
6. **Build CSR indices** — `csr_target` + edge properties, sorted by `(csr_source, csr_target)`. The source column is not stored; it's encoded by position in indptr
7. **Export** all tables as Parquet with `icebug_disk_version: v1` in KV metadata
8. **Generate** `schema.cypher` with LadybugDB-specific `WITH (storage = '', format = 'icebug-disk')` extensions

The in-memory variant (`icebug_format/memory.py:37-234`) follows the same CSR logic but operates on PyArrow tables via a transient DuckDB connection, returning an `IcebugMemGraph` dataclass with `src`, `dest`, `indices`, and `indptr` fields.

GraphAr support (`icebug_format/graphar.py:159-555`) reads GraphAr's YAML graph info and Parquet vertex/edge files, reassembles them, and feeds through the same CSR pipeline.

## Key Techniques

**CSR as the universal adjacency encoding.** Rather than storing edge lists or adjacency lists naively, every edge type gets its own CSR pair: `indptr` (length N+1, where N is the number of source nodes) and `indices` (length E, sorted by source then target). This gives O(1) neighbor lookup — `indptr[i]` through `indptr[i+1]` are node i's neighbors — and is the same representation used by sparse matrix libraries like SciPy and graph libraries like NetworkKit.

**Per-type dense ID mapping, not global.** Each node type (user, city, etc.) gets its own independent zero-based mapping, ordered by primary key. This means node IDs don't need to be globally unique or dense — only consistent within their type. The mapping tables (`{prefix}_mapping_{node_type}`) are preserved in the output so original IDs can be recovered.

**DuckDB as a portable SQL engine, not a dependency lock-in.** The spec (`doc/spec.md:486-507`) explicitly documents backend-agnostic requirements: any engine that can list tables, sort, join, compute cumulative sums, and write Parquet can reimplement the format. DuckDB is chosen for its embedded operation (no server), efficient Parquet I/O, and `unnest(range(...))` for generating node ID ranges.

**Schema-driven endpoint resolution.** An optional input `schema.cypher` with Cypher `CREATE REL TABLE` statements tells the converter which node types sit at each end of an edge. The parser handles backtick-quoted identifiers and case-insensitive matching. Without a schema, edges fall back to the first discovered node table for both endpoints.

**Column name aliasing with positional fallback.** Source columns match `source` → `src` → `from` → 0th column. Target columns match `target` → `destination` → `dest` → `to` → 1st column. This means the converter accepts input from diverse sources (Pandas, SQL exports, other graph tools) without requiring column renaming.

## Design Decisions

**Scan-optimized over write-convenient.** CSR + columnar Parquet is designed for a graph engine to mount files directly from S3-compatible storage and scan only the tables it needs. The trade-off: you can't incrementally update — every conversion is a full replacement that drops all existing output tables.

**SQL over bespoke graph code.** The entire conversion is expressed in SQL (CTEs, window functions, cumulative sums). This leverages DuckDB's battle-tested query optimizer rather than hand-rolling graph algorithms in Python. The cost is that the SQL is non-trivial to read and debug — the indptr construction alone spans a four-CTE query.

**Self-loops preserved in directed mode, deduplicated in symmetric.** In directed graphs, a `source = target` edge stays. With `--add-reverse-edges`, the reverse UNION ALL has a `WHERE source != target` clause to prevent self-loops from appearing twice. This is a practical choice for algorithms that need symmetric adjacency without double-counted self-connections.

**All-or-nothing reverse edges.** `--add-reverse-edges` applies to every edge type in the conversion. If you have a mix of symmetric (`friends`) and directed (`follows`) edges, you must run separate conversions or pre-process the data. This is a deliberate scope limitation, not an oversight — per-edge-type reverse control would require a more complex CLI interface.

**LadybugDB-specific Cypher extensions.** The generated `schema.cypher` includes `WITH (storage = '', format = 'icebug-disk')` — this tells LadybugDB (a columnar graph database) to query Parquet files in-place. It is not standard Cypher and is irrelevant to row-oriented graph databases like Neo4j. The format is designed for LadybugDB's architecture first, with portability as a secondary concern.

## Comparison Notes

Unlike [[Grist]] which combines spreadsheet and database, icebug is purely a format — it doesn't provide a query interface or UI. Unlike [[AntFly]] which is a full distributed search engine and graph database, icebug is the interchange layer that feeds into a graph engine. It's closer in spirit to Apache Parquet itself: a columnar layout that multiple engines can read, with a schema that describes how to interpret the files.

Compared to GraphAr (which icebug can ingest as input), icebug-disk simplifies the directory structure: one Parquet file per table rather than GraphAr's chunked property group directories. This makes it friendlier for direct S3 mounting where fewer files means fewer API calls.

The use of CSR for adjacency is shared with NetworkKit and SciPy sparse matrices, but icebug adds the property-graph layer on top: edge properties travel with the adjacency, and `schema.cypher` bridges the gap between raw CSR and typed property graphs.

---
*Sources: [[raw/icebug-format]]*
*Last updated: 2026-08-01*
