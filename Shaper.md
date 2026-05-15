# Shaper

Open-source, self-hosted data dashboards powered by DuckDB. Write SQL, get visualizations. All in SQL -- including chart type definitions via special casting syntax like `::BARCHART_STACKED`.

---

## Key Themes

#database #devtools #observability

Shaper's pitch is simple: SQL is the interface, DuckDB is the engine, and everything else is presentation. The SQL-first approach means anyone who can write queries can build dashboards, without learning a proprietary visualization language or dragging boxes around a canvas.

The embedded analytics story is where it gets interesting for product teams: white-labeling, row-level security via JWT, and JavaScript/React SDKs (no iFrames) make it plausible to embed Shaper dashboards directly into your product. The automated reporting (PDF, PNG, CSV, Excel on a schedule) covers the "just email me a report every Monday" use case that every BI tool needs.

DuckDB as the engine means it can query diverse data sources efficiently -- Parquet files, CSV, JSON, remote databases -- without requiring a central data warehouse. This is the "modern data stack in a box" approach.

Go + TypeScript implementation, Mozilla Public License 2.0.

## Critical Analysis

The SQL casting syntax for chart types (`::BARCHART_STACKED`) is clever but arguably a code smell -- it mixes presentation with query logic. For quick dashboards this is fine; for maintained analytics, you'd want the visualization definition separated. The DuckDB dependency is both a strength (great analytical performance, broad format support) and a constraint (DuckDB's SQL dialect, not always 1:1 with production databases). 1.1k GitHub stars suggests decent traction for a self-hosted tool. The Taleshape managed offering for regulated industries hints at the real business model.

---
*Sources: [[raw/shaper]]*
*Last updated: 2026-05-14*
