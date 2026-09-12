---
title: "sql-crack"
url: https://github.com/buva7687/sql-crack
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - databases-and-data
---

# SQL Crack: Visual SQL Query Analysis (VS Code Extension)

## Main Purpose
VS Code extension that transforms SQL queries into interactive visual flow diagrams, enabling developers to understand complex queries at a glance and track data lineage across entire SQL workspaces.

## Key Features

**Query Visualization:**
- Execution flow diagrams with color-coded operation nodes
- Multi-query support with tab navigation
- Column lineage tracing through JOINs, aggregations, and calculations
- CTE and subquery expansion via double-click
- Undo/redo layout history
- Query comparison mode (baseline vs. current)
- Complexity scores and performance analysis

**Workspace Analysis:**
- Graph view showing file and table relationships
- Lineage view with interactive data flow visualization
- Impact analysis for schema changes (MODIFY/RENAME/DROP)
- Cross-file dependency tracking

**Smart Analysis:**
- Quality warnings (unused CTEs, dead columns, duplicate subqueries)
- Performance hints (filter pushdown, join order, index suggestions)
- Performance scoring (0-100 based on anti-patterns)

**Interactive Features:**
- Bidirectional editor-to-diagram synchronization
- Keyboard navigation and search
- Multiple layout options (vertical, horizontal, compact, force, radial)
- Export to PNG, SVG, Mermaid.js, or clipboard

## Supported Dialects
MySQL, PostgreSQL, SQL Server, MariaDB, SQLite, Snowflake, BigQuery, Redshift, Hive, Athena, Trino, Oracle, Teradata

## Technical
TypeScript (99.8%), node-sql-parser, dagre layout engine. 100% local processing, no network calls, no telemetry.

Inspired by JSON Crack and Snowflake Query Profile.
