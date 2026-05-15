# sql-crack

VS Code extension that visualizes SQL queries as interactive execution flow diagrams. Column lineage tracing, CTE expansion, performance scoring, workspace-wide dependency analysis -- all running 100% locally with no telemetry.

---

## Key Themes

#devtools #database #observability

sql-crack brings visual query analysis from expensive database platforms into the editor. The execution flow diagrams with color-coded operation nodes make complex queries comprehensible at a glance -- especially joins, CTEs, and subqueries that are notoriously hard to reason about as flat text.

The workspace analysis features go beyond single queries: graph views of file and table relationships, cross-file dependency tracking, and impact analysis for schema changes (what breaks if I rename this column?). The quality warnings (unused CTEs, dead columns, duplicate subqueries) and performance hints (filter pushdown, join order, index suggestions) are the kind of analysis that usually requires running the query against a real database.

Supports 13 SQL dialects from MySQL to BigQuery to Teradata. The bidirectional editor-to-diagram synchronization means clicking a node in the diagram highlights the corresponding SQL, and vice versa.

Inspired by JSON Crack and Snowflake Query Profile.

## Critical Analysis

The privacy story (100% local, no network calls, no telemetry) is a significant differentiator -- many SQL analysis tools require sending queries to a cloud service, which is a non-starter for organizations with sensitive data. The breadth of dialect support is impressive for a VS Code extension. The node-sql-parser dependency means analysis is based on parsing rather than actual execution plans, which limits the accuracy of performance hints compared to tools that talk to a real database. But for understanding query structure and lineage, parsing is sufficient and much faster. The performance scoring (0-100 based on anti-patterns) is useful as a quick sanity check, though experienced DBAs will want real execution plans for optimization.

---
*Sources: [[raw/sql-crack]]*
*Last updated: 2026-05-14*
