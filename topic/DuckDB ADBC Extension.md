# DuckDB ADBC Extension

DuckDB gets a universal database connector via the ADBC (Arrow Database Connectivity) extension, enabling `read_adbc` and `ATTACH` to 30+ systems — Snowflake, Databricks, BigQuery, PostgreSQL, MySQL, and more — all through a single Arrow-native interface with connection pooling, metadata caching, and streaming bulk ingest. Built by Sam Arch and the Columnar Tech team, announced July 2026.

---

## What It Is

A DuckDB community extension that reverses DuckDB's ADBC role: previously, external systems reached into DuckDB via its ADBC driver. Now DuckDB reaches out to the entire ADBC ecosystem. Two primary interfaces:

- **`read_adbc('profile://name', query)`** — execute a remote query, get results as a DuckDB table
- **`ATTACH 'profile://name' AS alias (TYPE adbc)`** — mount a remote database and query it as if local, supporting SELECT, INSERT, COPY, and CTAS

Connection profiles are TOML files stored at OS-standard locations (`~/Library/Application Support/ADBC/Profiles/` on macOS). The `dbc` CLI installs ADBC drivers (`dbc install sqlite`, `dbc install postgres`, etc.).

## Why It Matters

This is the **JDBC moment for the Arrow ecosystem**. JDBC gave Java a universal database interface in 1997 and became database plumbing for 30 years. ADBC does the same for the columnar/Arrow world — but with zero-copy Arrow transfer instead of row-by-row serialization. DuckDB is the first major analytical database to adopt ADBC as a client, which matters because DuckDB has become the Swiss Army knife of data engineering (it's embedded in [[Streambed]] for query serving, powers [[Shaper]] dashboards, and replaces Athena in Lambda deployments as documented in [[Replace Athena with DuckDB (Lambda)]]).

The extension also solves a real coordination problem: the community can't build and maintain individual DuckDB connectors for every database. ADBC shifts the burden to the database vendors (who maintain their own ADBC drivers) and gives DuckDB users one interface to learn. This is the same argument that made ODBC/JDBC win, but applied to columnar analytics.

## Key Quotes

> "Fast, zero-copy data transfer" between column-oriented analytical databases — avoiding the slow column-to-row and row-to-column conversions required by legacy APIs like ODBC and JDBC.

The Arrow-native transfer path is the architectural win. ODBC/JDBC go through row-by-row serialization even when both sides are columnar. ADBC skips that. For analytical workloads moving millions of rows, this isn't a marginal improvement — it's the difference between wire-speed and an order of magnitude slower.

> The ADBC extension operates in autocommit mode.

The biggest production caveat. No multi-statement transactions yet. If you need atomicity across statements, you're not there. But the roadmap is explicit and the issues are filed — this is a v0.1 limitation, not a design choice.

> For INSERT or CTAS statements, rows are inserted in batches (roughly 2 million at a time by default) using ADBC's bulk ingest API.

The streaming bulk ingest is the feature that makes this production-grade for data movement. Without it, inserting a billion rows into Snowflake through DuckDB would OOM. With it, memory stays flat regardless of dataset size. Combined with the configurable `adbc_insert_buffer_size`, this is well thought-out for real workloads.

## Architecture Notes

**Predicate pushdown gap.** When using `ATTACH`, the extension currently fetches all columns and rows — DuckDB applies filters locally. The workaround is `read_adbc` with a hand-written query string, wrapped in a macro. Predicate/projection pushdown is the #1 and #2 GitHub issues. This is the right kind of v0.1 honesty: ship the thing, document what's missing, prioritize by what users actually need.

**Connection pooling and metadata caching** are table-stakes features that show the team understands production usage. Without pooling, every SQL statement opens and closes a connection — unbearable for interactive use. Without caching, `SHOW TABLES` hits the remote every time. Both are configurable and have clear cache invalidation (`CALL adbc_clear_cache()`).

**The `dbc` CLI** is a smart sidecar. Managing ADBC drivers and connection profiles is the kind of friction that kills adoption. `dbc install postgres` → write a TOML → query. The profile-per-connection model means credentials stay in files (not SQL strings), which is correct but means you still need a secrets story for production.

## Critical Analysis

**The universal connector bet is right, and early.** ADBC has ~30 drivers today. That's enough to be useful but not enough to be universal — Oracle, SQL Server, and Redshift have drivers, but coverage is uneven. The bet is that ADBC becomes the standard (like JDBC did), and DuckDB riding that wave early gives it a connectivity story that no other embedded analytical database has. If ADBC doesn't gain traction, this extension becomes a well-engineered cul-de-sac.

**DuckDB as universal data router.** Between this extension, the existing Postgres scanner, SQLite scanner, MySQL scanner, and Snowflake scanner, DuckDB is positioning itself as the universal query federation layer. You don't move data into DuckDB — you point DuckDB at data wherever it lives. Combined with DuckDB's Parquet/CSV/JSON reading and writing, you get a single binary that can extract from anywhere, transform in-process, and load anywhere. This is ETL without the ELT platform.

**The Columnar Tech team is building an ecosystem, not just an extension.** They built `dbc` (the ADBC driver manager), the extension itself, and agent skills for AI coding tools. They're contributing the extension back to the Apache Arrow project. This is a company aligning its commercial interests with open-source ecosystem growth — a healthy pattern when done transparently.

**The agent skills angle is interesting but early.** Installing skills via `gh skill install columnar-tech/skills` lets AI coding agents set up ADBC connections. This fits the pattern of agents-as-DB-clients ([[DAB]] does this via MCP). But the real test is whether an agent can debug a failed ADBC connection — the happy path is easy, the failure modes are where agents struggle with database tooling.

**Comparison to DAB.** [[DAB]] (Microsoft's Data API Builder) auto-generates REST/GraphQL/MCP endpoints over databases. ADBC is a lower-level protocol — it's the transport, not the API layer. DAB could theoretically speak ADBC underneath. They're complementary: ADBC for direct analytical querying, DAB for exposing databases to applications and agents.

**What's missing for production:**
1. **Transactions.** Autocommit-only is fine for analytics, a problem for operational workloads.
2. **Predicate pushdown for ATTACH.** Without it, `SELECT * FROM billion_row_table WHERE id = 42` pulls the whole table. Fixed by using `read_adbc` instead, but the ergonomic gap matters.
3. **Credential management.** TOML files work for development. For production, you'll need env vars, secrets managers, or OAuth — none of which are mentioned.
4. **Query cancellation and timeout.** No mention of what happens when a remote query runs for 10 minutes.

## Themes

#database #connector #arrow #duckdb #adbc #tool #data-engineering #interoperability

## Related Pages

- [[Streambed]] — embeds DuckDB as a query server for Iceberg tables; ADBC would let it federate queries outward
- [[Databases and Data]] — hub page; ADBC is a new entry in the "convergence through connectivity" pattern
- [[DAB]] — Microsoft's API layer over databases; complementary to ADBC's lower-level transport
- [[FlareDB]] — Arrow-backed database tables; the other side of the Arrow-native coin
- [[Replace Athena with DuckDB (Lambda)]] — DuckDB as cost-killer for cloud analytics; ADBC extends this to any database
- [[Shaper]] — DuckDB for dashboards; ADBC means Shaper could pull from Snowflake/BigQuery without ETL
- [[AliSQL]] — grafts DuckDB onto MySQL; ADBC would let DuckDB query AliSQL tables without grafting

---
*Sources: [[raw/duckdb-adbc-extension]]*
*Last updated: 2026-07-11*
