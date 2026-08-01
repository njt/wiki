---
url: https://github.com/Ladybug-Memory/icebug-format
title: icebug-format
author: Ladybug Memory
date_fetched: 2026-08-01
date_published: 2025
---

# icebug-format

A standardized graph format for efficient graph data interchange. Converts property graphs stored as relational tables into CSR (Compressed Sparse Row) adjacency format, backed by Parquet files (disk) or Apache Arrow tables (memory).

## Repository Structure

```
icebug-format/
  icebug-format.py           # CLI entry point (7 lines)
  icebug_format/
    __init__.py              # Exports main() and IcebugMemGraph
    cli.py                   # CLI: DuckDB→CSR conversion (866 lines)
    memory.py                # In-memory IcebugMemGraph class (234 lines)
    graphar.py               # GraphAr→CSR converter (607 lines)
    test_csr_duckdb.py       # Verify CSR disk output (155 lines)
  tests/
    test_memory.py           # Tests for IcebugMemGraph (285 lines)
    test_cli.py              # Tests for CLI (447 lines)
  doc/
    spec.md                  # CLI specification (507 lines)
  examples/
    karate/                  # Zachary's karate club example graph
  pyproject.toml             # Project config (v1.0.0, Python 3.9+)
```

Total: ~3,328 lines across source and tests.

## Two Formats

### icebug-disk (Parquet + DuckDB)

The CLI `icebug-format` converts a DuckDB source database containing `nodes_*` / `edges_*` tables into:
- `nodes_<name>.parquet` — original node data
- `indices_<name>.parquet` — target node for each edge, sorted by source
- `indptr_<name>.parquet` — row-pointer array (size N+1)
- `schema.cypher` — Cypher schema for mounting in LadybugDB

All Parquet files include `icebug_disk_version: v1` in their KV metadata.

### icebug-memory (Arrow + in-memory CSR)

The Python API `IcebugMemGraph.from_arrow_tables()` converts Arrow tables directly into an in-memory CSR graph:
- `src` / `dest` — node tables passed through unchanged
- `indices` — target column + edge properties, ordered by (source, target)
- `indptr` — ptr column of length len(src) + 1

## Architecture Pattern

The project is a **format converter with a pipeline architecture**: read relational tables → create dense node mappings → join edges to mappings → build CSR indptr/indices → export to Parquet → generate Cypher schema.

It does not store graphs itself; it produces a representation that graph engines can mount directly from object storage without ingesting into a proprietary store.

## Key Technical Decisions

### Dense integer node IDs via per-type mapping

Each node type gets its own zero-based dense mapping (`csr_index`), ordered deterministically by the node table's first column. This means node IDs don't need to be globally dense or globally unique — only per-type. The mapping tables are stored as `{prefix}_mapping_{node_type}`.

### CSR (Compressed Sparse Row) as the adjacency encoding

All adjacency is stored as CSR — the standard format for sparse matrices. `indptr[i]` is the start offset in `indices` for source node `i`, and `indptr[i+1] - indptr[i]` gives the out-degree. This enables O(1) lookup of a node's neighbors. The `csr_source` column is not stored in the final indices table — it's encoded by the offsets.

### DuckDB as the SQL engine

The implementation uses DuckDB for all heavy lifting: table discovery, sorting, JOIN operations for endpoint mapping, GROUP BY for degree computation, and Parquet export. DuckDB's `unnest(range(...))` is used to generate node ranges for the indptr computation, and `row_number() OVER (...)` for dense ID assignment. The spec is explicitly backend-agnostic, documenting requirements a non-DuckDB backend must meet.

### Self-loop handling

In directed mode, self-loops are preserved as-is. With `--add-reverse-edges`, self-loops appear only once (forward only) — the `WHERE source != target` clause on the reverse UNION ALL prevents duplication.

### Reverse-edge expansion

`--add-reverse-edges` emits the reverse direction for each non-self-loop edge, producing symmetric adjacency. Edge properties are copied onto both directions. This is all-or-nothing per conversion — cannot be applied selectively per edge type. Requires the same node table on both sides of every edge.

### Column name resolution

Source and target columns are resolved by priority-order alias matching with a positional fallback:
- Source: `source` → `src` → `from` → 0th column
- Target: `target` → `destination` → `dest` → `to` → 1st column

Any remaining columns become edge properties.

## Implementation Details

### CLI conversion flow (cli.py:create_csr_graph_to_duckdb)

1. ATTACH source DB, discover `nodes_*` and `edges_*` tables via `information_schema.tables`
2. Parse optional `schema.cypher` for edge endpoint type info
3. Copy node tables to output, sorted by primary key
4. For each edge table:
   - Build inline CTE mappings (`row_number() OVER (ORDER BY pk) - 1`)
   - Join edge source/target to mapping tables (inner join — dangling edges dropped)
   - Optional reverse-edge expansion via UNION ALL
   - Build indptr: cumulative degree per source node, prepended with 0
   - Build indices: `csr_target` + edge properties, ordered by (source, target)
   - Drop temporary relations table
5. Export all tables to Parquet with `icebug_disk_version` metadata
6. Generate `schema.cypher` with `WITH (storage = '', format = 'icebug-disk')` extensions

### In-memory conversion flow (memory.py:IcebugMemGraph.from_arrow_tables)

1. Resolve source/target column names from relationship schema
2. Extract edge property columns (everything except source/target)
3. Build DuckDB CTEs for dense ID mapping
4. Construct forward and (optionally) reverse SELECT queries
5. Register Arrow tables with DuckDB, execute relations query
6. Build indptr via cumulative degree computation
7. Build indices sorted by (csr_source, csr_target)
8. Return IcebugMemGraph dataclass

### GraphAr support (graphar.py)

Reads GraphAr's YAML-based graph info, parses Parquet vertex and edge files from the GraphAr directory layout, reassembles vertices and edges, then feeds through the same CSR conversion pipeline. GraphAr's edge direction and property structure map naturally onto icebug's CSR representation.

## Dependencies

- `duckdb>=1.3.2` — SQL execution, sorting, Parquet export
- `pyarrow>=21.0.0` — Arrow table handling, Parquet metadata
- `graphar` (optional) — GraphAr format input

## Design Trade-offs

**Optimized for**: Scan efficiency from object storage. CSR + Parquet means a graph engine can read just the files it needs without ingesting the whole graph.

**Sacrificed**: Write-time convenience. The format is a conversion target, not something you author directly. You need your data in DuckDB (or GraphAr) first.

**Missing**: Incremental updates. The CLI drops all existing output tables before conversion — full replacement only.

**Test mode limitation**: `--test` with `--limit N` applies `floor(N / num_edge_tables)` per table before ordering, but ordering before the LIMIT is backend-dependent. Test-mode samples are not deterministic across backends.

## Edge Cases Covered

- Source/target column aliases (6 different naming conventions)
- Backtick-quoted identifiers in Cypher schema
- Column names with spaces (properly quoted in SQL)
- Empty edge tables (produces zero-length indptr)
- Self-loops: preserved in directed mode, deduplicated in reverse mode
- Heterogeneous edges (different node types on each end, via schema.cypher)
- Missing schema.cypher (falls back to first node table for both endpoints)
