---
url: https://www.rilldata.com/blog/introducing-metrics-sql-a-sql-based-semantic-layer-for-humans-and-agents
date_fetched: 2026-07-05
backfilled: true
---

# Introducing Metrics SQL: A SQL-based semantic layer for humans and agents

SQL has been the lingua franca of data for over 40 years. We decided to make it the lingua franca of our metrics-based semantic layer too.

When we built Rill, we made a bet that metrics – concepts like revenue, MAU, ROAS — are the core primitive for semantic layers.

We also believed that querying a semantic layer (and its metrics) should not require learning a custom language or API. Instead, the semantic layer should speak SQL, the language every database, BI tool, and AI agent already knows.

This blog post is a deep technical dive into how Rill's Metrics SQL works, why we built it the way we did, and where we're taking it next.

##  **Using SQL and metrics for a determistic, secure, and fast semantic layer** 

In a modern analytics stack, a business metric like revenue per active user needs to be defined in multiple places: a dbt model, a Looker Explore, a Metabase question, a Python notebook, a Slack bot and in your AI Agent metadata. Over time, each definition drifts from the others. When finance asks "why does your number not match mine?", the answer is usually "we're computing it differently."

The metrics layer concept — define business logic once, query it anywhere — has been around for years. But implementations consistently run into the same trap: they invent a new query language. GraphQL queries, proprietary MDX expressions, or JSON-over-REST APIs, each requires learning a new abstraction. Each breaks existing SQL tooling. Each becomes a barrier between business logic and applications.

Julian Hyde, one of the creators of  Apache Calcite  and a pioneer of the SQL ecosystem, argued in his Data Council  talk  that metrics need first-class SQL representation — not because SQL is perfect, but because it is universal: the one language every tool, every engineer, and every AI model already speaks. Julian proposed adding a first-class `MEASURE` semantics in SQL — where aggregates are declared as semantic objects, not inline expressions, and the database engine handles computing measures in the correct context.

That framing resonated and we built a metrics-first semantic layer queryable using SQL. Here is what that enabled:

- *A deterministic source of truth.*Every consumer — dashboards, notebooks, APIs, AI agents — resolves a metric from the same governed definition. No copies, no drift. An agent querying the- `revenue`metric returns the same answer every time, not an inferred expression on raw table schemas.
- *A universal interface with access policies.*SQL works for human analysts and LLMs alike. No new language to learn, no model fine-tuning required. Row-level security and access rules can be defined once on this interface, and enforced on every query regardless of who or what is asking.
- *A higher performance architecture.*Extending SQL with metrics creates a path for SQL-based optimizations: materialized views can serve metrics queries rather than raw fact tables, improving performance and cost. Database Indexes and Projections can be intelligently tuned based on query patterns.

##  **Rill's metrics layer is composed of measures and dimensions** 

What is a metric? In Rill, a metric is defined as an aggregate measure expression — such as total revenue, average session duration, or return on advertising spend (ROAS) — evaluated in a dimensional context. As illustrated below, a revenue metric can be filtered to a specific geo, product category, and time dimension. This mirrors a well-worn concept in business intelligence systems: an OLAP cube.

Dimensions are the attributes you slice and group by: geography, product category, channel, date. In SQL terms, they appear in `GROUP BY`. A dimension can be a raw column (`country`) or a computed expression (`dictGet('category_dict', 'product_category', product_id)`).

Along with Measures and Dimensions, Rill metrics layer also allow you to define additional metadata properties such as:

- AI Instructions - Any additional business context other than the metadata that can be useful for AI agents, e.g. Definition of Financial Year beginning and end.
- Timeseries columns and an optional smallest time grain

## Metrics SQL and its transpilation to OLAP SQL

Metrics SQL is a SQL dialect that lets you query a metrics view as if it were a table — selecting dimensions and measures by name, filtering, ordering, and limiting results, without writing any aggregation logic in the query. Metrics SQL allows a user to express business logic such as top lists, dimensional filters, time range queries and comparisons in far less complexity and fewer tokens.

A Metrics SQL query passes through three logical layers before reaching the OLAP engine.

- Parser parses the query and validates its syntax
- Query Compiler resolves each name against the metrics view definition classifies measures and dimensions, adds inferred group by.
- Executor applies security filters and any semantic rewrites before handing a fully-formed, parameterized SQL string to the OLAP engine.

##  **Defining a metrics view** 

Rill metric view definitions use SQL embedded within YAML, where YAML provides the structural envelope and SQL provides the expression-level logic embedded within it. Below is an example of a metrics_view definition.

```
# revenue_metrics.yaml
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
##  **Concrete examples: Metrics SQL → database SQL** 

Here are real transformations, showing exactly how Metrics SQL maps to the underlying SQL from OLAP engines. In these examples, we use ClickHouse as our target database engine, but notably Metrics SQL supports DuckDB, Snowflake, and Druid SQL dialects among others.

###  **Example 1: Top 10 countries by revenue** 

**Metrics SQL:**

`SELECT country, revenue FROM revenue_metrics ORDER BY revenue DESC LIMIT 10`**Output SQL:**

```
SELECT
  ("country") AS "country",
  sum(order_usd) AS "revenue"
FROM "revenue_model"
GROUP BY 1
ORDER BY "revenue" DESC NULLS LAST
LIMIT 10
```
A few things to notice in the output:

- The `FROM`clause is rewritten to the underlying table.
- The `revenue`measure is expanded to its aggregate expression`sum(order_usd)`.
- The `GROUP BY`is inferred from the set of selected dimensions.

###  **Example 2: Filter on a computed dimension** 

**Metrics SQL:**

```
SELECT country, revenue
FROM revenue_metrics
WHERE product_category = 'Electronics'
ORDER BY revenue DESC
LIMIT 10
```
**ClickHouse SQL:**

```
SELECT ("country") AS "country", sum(order_usd) AS "revenue"
FROM "revenue_model"
WHERE (dictGet('category_dict', 'product_category', product_id) = ?)
GROUP BY 1
ORDER BY "revenue" DESC NULLS LAST
LIMIT 10
-- args: ["Electronics"]
```
In this example `product_category` is a computed dimension; the compiler expands its `dictGet` expression transparently in the filter. The literal `'Electronics'` becomes a parameterized argument, ensuring SQL injection safety regardless of where the query originates.

### Example 3: Subquery with filter

Metrics SQL supports subquery with filters: the outer query can filter a dimension using a set of values produced by an inner query. The inner query can itself use `HAVING` to filter on a measure. This lets you express patterns like "show me order volume for countries where total revenue exceeded a threshold."

**Metrics SQL:**

```
SELECT country, order_volume
FROM revenue_metrics
WHERE country IN 
  (SELECT country 
   FROM revenue_metrics 
   HAVING revenue > 10000)
```
**ClickHouse SQL:**

```
SELECT ("country") AS "country", (count(distinct order_id)) AS "order_volume"
FROM "revenue_model"
WHERE ("country") IN (
  SELECT "country"
  FROM (
    SELECT ("t1"."country") AS "country", ("t1"."revenue") AS "revenue"
    FROM (
      SELECT ("country") AS "country", (sum(order_usd)) AS "revenue"
      FROM "revenue_model"
      GROUP BY 1
    ) t1
    WHERE (("t1"."revenue") > ?)
  )
)
GROUP BY 1
-- args: [10000]
```
The inner `HAVING revenue > 10000`  makes the aggregation move into an inner subquery and the filter becomes a `WHERE` on the outer level. The outer query then uses that result set as an `IN` list to filter the dimension in the main query.

###  **Example 4: Dynamic time range expressions** 

**Metrics SQL:**

```
SELECT country, revenue
FROM revenue_metrics
WHERE order_date > time_range_start('7D as of watermark')
  AND order_date <= time_range_end('7D as of watermark')
```
**ClickHouse SQL:**

```
SELECT
  ("country") AS "country",
  (sum(order_usd)) AS "revenue"
FROM "revenue_model"
WHERE (("order_date") > ? AND ("order_date") <= ?)
GROUP BY 1
-- args: [2025-03-29T00:00:00Z, 2025-04-05T00:00:00Z]
```
The `time_range_start` and `time_range_end` functions accept Rill's  time syntax  expressions that support watermark-relative offsets and grain-alignment operators. The compiler resolves these against the metrics view's data watermark at parse time, and the resulting timestamps become parameterized arguments.

###  **Example 5: Window function measure** 

Window function measures require a time dimension to anchor the frame. The measure `revenue_7day_avg` is defined in the metrics view as:

```
	measures: 
  - name: revenue_7day_avg
    display_name: 7-Day Rolling Average Revenue
    expression: AVG(revenue)
    requires: [revenue]
    window:
      order: order_date
      frame: RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
```
**Metrics SQL:**

`SELECT order_date, revenue_7day_avg FROM revenue_metrics`**ClickHouse SQL:**

```
SELECT ("t1"."order_date") AS "order_date",
       (AVG(revenue) OVER (
         ORDER BY "t1"."order_date"
         RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
       )) AS "revenue_7day_avg"
FROM (
  SELECT ("order_date") AS "order_date", (sum(order_usd)) AS "revenue"
  FROM "revenue_model"
  GROUP BY 1
) t1
```
The metrics layer produces a two-level query: the inner subquery computes `revenue` as a base aggregate grouped by date, and the outer query applies the window function over those results. The `requires` field tells the executor which base measure to include in the inner query even if the user didn't explicitly select it.

##  **Querying metrics SQL with our CLI, HTTP, and via Claude** 

Rill exposes multiple ways of querying metrics using Metrics SQL.

**1. CLI**: Rill exposes Metrics SQL directly from the command line. With a project running locally:

```
# Start your local Rill project
rill start
# Query a metrics view using Metrics SQL
rill query --local \
  --resolver metrics_sql \
  --properties sql="<Metrics SQL query>"
```
Targeting a Rill Cloud project works identically, substituting the project name:

```
rill query --project my-project \
  --resolver metrics_sql \
  --properties sql="<Metrics SQL query>"
```
2. **HTTP API**: Rill exposes a built-in HTTP endpoint:

```
curl -X POST "https://admin.rilldata.com//v1/orgs/{org}/projects/{project}/runtime/api/metrics-sql" \
  -H "Authorization: Bearer $RILL_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"sql": "<A valid metrics SQL>"}'
```
##  **Querying metrics from AI agents: Claude/ChatGPT** 

Rill ships a Model Context Protocol (MCP) server that exposes your metrics layer directly to AI agents. Instead of pointing Claude at your raw data warehouse, you connect it to Rill — and every answer it gives is backed by the same governed measures and security policies as your dashboards.

There are two ways to use AI with Rill's Metrics layer:

- **Rill's built-in AI Chat**: It is available as part of your dashboards and doesn't require any additional setup. Open your project, click the AI tab, and ask questions in plain English. Rill handles the Metrics SQL generation, and the query execution — you get governed, accurate answers directly in the browser.
- **Connecting to Claude using MCP**: Claude can connect to any Rill project via MCP. Instructions to connect and configure your them can be found in the docs here . Once connected, Claude can browse your metrics views, discover dimensions and measures, and answer questions like- *"What are my week-on-week changes in revenue by product category?"*with results grounded in your defined business logic.

You can also add `ai_instructions` to your metrics view YAML to give Rill additional context about how measures should be interpreted, improving answer quality for domain-specific questions.

##  **Current limitations** 

Metrics SQL is intentionally a restricted subset of SQL. These are design constraints, not oversights — they keep the compilation model tractable and the security guarantees auditable.

- No JOINs across metrics views. Each Metrics SQL query targets exactly one metrics view. Cross-view analysis requires defining a joined or denormalized model at the semantic layer.
- No `SELECT *`. Dimensions and measures must be named explicitly. This is both a governance mechanism and a performance optimization — OLAP queries should never scan unnecessary columns.
- Measure filters must use `HAVING`, not`WHERE`.`WHERE`filters are applied pre-aggregation against the underlying table — only dimension filters are valid there. Filtering on a measure (e.g.,`total_spend > 10`) must use`HAVING`.
- MetricsSQL is constantly evolving and doesn't support all operators/expressions yet.

##  **Future: Semantic pushdown, pushing metrics into the database layer** 

The current architecture compiles Metrics SQL in Rill's runtime layer and generates native OLAP SQL. This is powerful, but it requires all queries to route through Rill's runtime.

The longer-term vision is for OLAP engines to speak metrics semantics natively. If databases like DuckDB, ClickHouse, Snowflake understood `MEASURE` as a first-class SQL keyword, a metrics view could be a native database object stored alongside existing table metadata, and all compilation and query optimizations would move into the engine itself.

This would enable:

- **Zero-hop metrics queries**: Tools like- `psql`could query metrics directly from database without any middleware
- **Database-level optimization**: The database could optimize across the measure expansion and the outer query as a single plan. The engine can make better use of table stats to do more specific query optimizations.

Rill's Metrics SQL layer is our implementation of this vision today — designed so that when engines adopt richer metrics semantics natively, the interface stays identical and only the compilation target changes.

##  **Getting started** 

Metrics SQL is available in Rill Developer (local) and Rill Cloud. Define a metrics view, then query it immediately from the CLI, the HTTP API, or any connected BI tool.

```
# Install Rill
curl -s https://cdn.rilldata.com/install.sh | bash
# Start your project
rill start
```
The full Metrics SQL reference — including all supported filter functions, time range expressions, and the security model — is in the Rill documentation .
