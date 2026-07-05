---
url: https://www.rilldata.com/blog/introducing-metrics-sql-a-sql-based-semantic-layer-for-humans-and-agents
title: "Introducing Metrics SQL: A SQL-based semantic layer for humans and agents"
author: Nishant Bangarwa
date_fetched: 2026-05-15
date_published: 2026-04-08
---

# Introducing Metrics SQL: A SQL-based semantic layer for humans and agents

**Author:** Nishant Bangarwa (Rill Data)
**Published:** April 8, 2026
**Reading time:** 5 minutes

## Full Content

The article announces Rill Data's Metrics SQL, a SQL dialect that lets users query a metrics view (pre-defined measures and dimensions) as if it were a table. The thesis: metrics (revenue, MAU, ROAS) are the core primitive for semantic layers, and SQL is the right language for querying them — not a proprietary DSL.

### Background & Motivation

The problem: a business metric like "revenue per active user" gets defined in dbt, Looker, Metabase, Python notebooks, Slack bots, and AI agents — and each definition drifts. The solution is a metrics layer that enforces one definition, queried anywhere.

The approach credits Julian Hyde (creator of Apache Calcite), who argued at Data Council that metrics need first-class SQL representation via a `MEASURE` keyword where "aggregates are declared as semantic objects" rather than inline expressions.

Three claimed benefits:
1. **Deterministic source of truth** — every consumer resolves metrics from the same governed definition; "no copies, no drift"
2. **Universal interface with access policies** — SQL works for both humans and LLMs; row-level security enforced once across all query sources
3. **Higher performance architecture** — materialized views can serve metrics queries; indexes/projections tuned based on query patterns

### Definition Format

Metrics are defined in YAML with embedded SQL expressions. A metric is "an aggregate measure expression evaluated within a dimensional context" (OLAP cube concept).

```yaml
type: metrics_view
model: revenue_model
timeseries: order_date
smallest_time_grain: hour
measures:
  - name: revenue
    expression: sum(order_usd)
    display_name: Total Revenue
    format_preset: currency_usd
  - name: order_volume
    expression: count(distinct order_id)
    display_name: Order Volume
    format_preset: humanize
dimensions:
  - name: country
    column: country
    display_name: Country
  - name: product_category
    expression: dictGet('category_dict', 'product_category', product_id)
    display_name: Product Category
```

Measure metadata includes AI Instructions (business context for agents), timeseries columns, and optional smallest time grain.

### Transpilation Architecture

Three logical layers:
1. **Parser** — validates syntax
2. **Query Compiler** — resolves names against the metrics view definition, classifies measures/dimensions, adds inferred GROUP BY
3. **Executor** — applies security filters and semantic rewrites, then hands a fully-formed parameterized SQL string to the OLAP engine

Supported engines: ClickHouse, DuckDB, Snowflake, Druid.

### Key Transformations (with examples)

1. **Measure expansion**: `revenue` → `sum(order_usd)`, `FROM` rewritten to underlying table, GROUP BY inferred
2. **Filter on computed dimensions**: `WHERE product_category = 'Electronics'` → `WHERE dictGet(...) = ?` — parameterized for SQL injection safety
3. **HAVING → WHERE transformation**: `HAVING revenue > 10000` in subquery becomes `WHERE` on inner subquery's outer level
4. **Dynamic time ranges**: `time_range_start('7D as of watermark')` — resolved against data watermark at parse time
5. **Window function measures**: `requires` field tells executor which base measure to include in inner query even if user didn't explicitly select it

### Query Interfaces

- CLI (local and cloud): `rill query --resolver metrics_sql --properties sql="<query>"`
- HTTP API: POST to `/v1/orgs/{org}/projects/{project}/runtime/api/metrics-sql`
- MCP server: exposes metrics layer to AI agents; agents browse metrics views, discover dimensions/measures, answer questions grounded in defined business logic

### Current Limitations

- No JOINs across metrics views (each query targets exactly one metrics view)
- No `SELECT *` (must name dimensions/measures explicitly — "both a governance mechanism and a performance optimization")
- Measure filters must use `HAVING`, not `WHERE` (WHERE applies pre-aggregation at the table level)
- Evolving — doesn't support all operators/expressions yet

### Future Vision: Semantic Pushdown

The longer-term goal is for OLAP engines to natively support `MEASURE` as a first-class SQL keyword, enabling "zero-hop metrics queries" where tools like `psql` query metrics directly from the database with no middleware.

Metrics SQL is "designed so that when engines adopt richer metrics semantics natively, the interface stays identical and only the compilation target changes."
