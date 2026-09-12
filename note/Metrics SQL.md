# Metrics SQL

Rill Data's SQL dialect for querying a metrics-based semantic layer. Instead of a proprietary DSL, they bet that SQL is the right interface for both humans and AI agents to query governed business metrics (revenue, MAU, ROAS). YAML defines measures and dimensions; Metrics SQL queries them as if they were a table, then a transpiler expands everything into engine-native SQL with inferred GROUP BY, security filters, and parameterized literals.

---

## Key Quotes

> "SQL has been the lingua franca of data for over 40 years. We decided to make it the lingua franca of our metrics-based semantic layer too."

The core bet: don't invent a new query language when SQL already works and every data tool speaks it. This is [[Smart Models Dumb Pipes]] applied to analytics — SQL is the dumb pipe, metrics definitions are the smart model.

> "Over time, each definition drifts from the others."

The diagnosis that motivates the entire product. This is the same drift problem that [[Long Live Systems of Record]] identifies: when "where does the truth live" isn't settled, every consumer creates their own truth.

> "Measures are declared as semantic objects" — Julian Hyde, creator of Apache Calcite

The lineage matters. Hyde's argument (at Data Council) was that aggregates should be first-class SQL citizens rather than inline expressions. Rill's Metrics SQL is the nearest implementation of that vision, even though no OLAP engine supports `MEASURE` natively yet.

> "No JOINs across metrics views. Each query targets exactly one metrics view."

Honest about limitations. This constraint is both a simplification (one cube per query) and a real restriction — real-world analytics often wants to join metrics from different business domains. The bet is that governance and determinism are worth the tradeoff.

---

## Key Themes

- #tool — Rill Data's semantic layer product
- #concept — Metrics as first-class SQL objects; transpilation as deterministic compilation
- #pattern — YAML definitions + SQL queries = governed analytics; the MCP server pattern for agent access
- #comparison — vs. dbt (transforms, not a semantic layer), vs. Looker (LookML, proprietary), vs. Cube (YAML + REST/GraphQL, not SQL-native)

---

## Critical Analysis

**The good:** Making SQL the interface rather than inventing yet another query language is the right call. Every BI tool, notebook, and AI agent already speaks SQL. The transpilation approach — parsing Metrics SQL, resolving against a metrics view, producing engine-native SQL — is a compiler, not a black box. That means the output is inspectable, debuggable, and portable across OLAP engines. The MCP server integration is forward-looking: agents don't need a custom API, they just need to discover what measures and dimensions exist and query them in SQL.

**The tension:** This is an intermediate solution by design. The `MEASURE` keyword doesn't exist in any major OLAP engine's SQL dialect yet. So Metrics SQL is a transpiler that simulates what engines should do natively. The article is refreshingly honest about this — "designed so that when engines adopt richer metrics semantics natively, the interface stays identical and only the compilation target changes." But it means Rill is betting on engine vendors adopting `MEASURE` semantics, which may or may not happen. If they don't, Metrics SQL remains middleware forever.

**The missing piece:** The "one metrics view per query, no JOINs" constraint is the real limitation. Business questions often span metrics views — "show me revenue per customer alongside support ticket volume." Solving that requires either denormalizing everything into one view (defeating the purpose) or building a cross-view query layer that understands how metrics relate. The article doesn't address this.

**Bottom line:** A well-argued approach to a real problem. The SQL-native bet differentiates it from Cube and Looker. The MCP integration makes it agent-native. The limitations are honestly stated. Whether the OLAP ecosystem converges on `MEASURE` semantics will determine if this is a stepping stone or permanent middleware.

---

*Sources: [[summary/introducing-metrics-sql]]*
*Last updated: 2026-05-15*
