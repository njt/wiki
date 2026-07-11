---
url: https://columnar.tech/blog/announcing-duckdb-adbc-extension/
title: Announcing the DuckDB ADBC Extension
author: Sam Arch
date_fetched: 2026-07-11
date_published: 2026-07-08
---

## TL;DR

The ADBC extension enables DuckDB to connect with Snowflake, Databricks, BigQuery, PostgreSQL, MySQL, and any other system offering an ADBC driver. Built on Apache Arrow, it allows "fast data transfer to and from column-oriented databases" and works across the Arrow ecosystem. Installation is done via `INSTALL adbc FROM community;` followed by `LOAD adbc;`.

## The Missing Link Between DuckDB and External Databases

DuckDB has grown rapidly thanks to its usability and speed, but since not all data lives inside it, the system needs ways to communicate with external sources. The community previously built vendor-specific extensions for systems like SQLite, PostgreSQL, and Snowflake. However, each of these brings "another interface, feature set, and collection of quirks for DuckDB users to learn." Moreover, many widely used systems like Databricks, Redshift, and Oracle lack dedicated DuckDB extensions, as building and maintaining individual connectors for each system exceeds what the community can sustain.

## Enter ADBC

ADBC (Arrow Database Connectivity) serves as a universal data access API supporting more than 30 query systems across several categories:

- **Transactional databases:** PostgreSQL, MySQL, MariaDB, SQLite, Oracle Database, Microsoft SQL Server, CockroachDB, YugabyteDB, TiDB
- **Analytical databases:** DuckDB, ClickHouse, Exasol
- **Data warehouses:** Snowflake, Databricks, BigQuery, Amazon Redshift, Teradata
- **Lakehouse engines:** Trino, Dremio, StarRocks, Apache Doris
- **Time-series databases:** InfluxDB, TimescaleDB, GreptimeDB

Built on Apache Arrow—an efficient columnar data format widely supported by modern data systems—ADBC provides two major benefits: "fast, zero-copy data transfer" between column-oriented analytical databases (avoiding the slow column-to-row and row-to-column conversions required by legacy APIs like ODBC and JDBC), and smooth interoperability across the growing Arrow-compatible ecosystem.

## The ADBC Extension for DuckDB

DuckDB already participates in the ADBC ecosystem from the other direction—its shared library functions as an ADBC driver (installable via `dbc install duckdb`), and a separate ADBC driver exists for Quack, DuckDB's client-server protocol. These join the many existing ways DuckDB works with Apache Arrow.

This new ADBC extension reverses the connection flow: rather than external systems reaching into DuckDB, DuckDB now reaches out across the entire ADBC ecosystem. Users can query ADBC databases with `read_adbc` or use `ATTACH` to connect to an ADBC database and run "SELECT, INSERT, COPY, and CTAS statements as if it were local to DuckDB."

The author credits Rusty Conover for first connecting DuckDB outward to the ADBC ecosystem via his `adbc_scanner` extension. Their approach differs by offering easier integration with ADBC connection profiles, broader database compatibility, and "automatic connection pooling, automatic metadata caching, and memory-efficient INSERT and CTAS statements through streaming bulk ingest." They plan to contribute this open source extension to the Arrow project as an official ADBC library.

## What the ADBC Extension Can Do

### Install an ADBC Driver and Create a Connection Profile

The article demonstrates using a SQLite `games` database, downloadable from `https://data.columnar.tech/games.sqlite`. Users install the ADBC driver via `dbc`, the tool built by the team:

```sh
dbc install sqlite
```

A connection profile is a TOML file storing database connection info. A profile named `mydb.toml` for the SQLite games database looks like:

```toml
profile_version = 1
driver = "sqlite"

[Options]
uri = "./games.sqlite"
```

Users place this profile in the default ADBC profiles directory for their OS:

- Linux: `~/.config/adbc/profiles/`
- macOS: `~/Library/Application Support/ADBC/Profiles/`
- Windows: `%LOCALAPPDATA%\ADBC\Profiles\`

### Read Data with `read_adbc`

The simplest usage calls `read_adbc` with a profile URI and SQL query, executing it remotely and returning results to DuckDB:

```sql
INSTALL adbc FROM community;
LOAD adbc;
SELECT * FROM read_adbc('profile://mydb', 'SELECT * FROM games');
```

This returns a table showing five classic board games—Monopoly, Scrabble, Clue, Candy Land, and Risk—with their IDs, names, inventors, years, age ranges, player counts, and list prices. The returned data functions as a regular DuckDB table, ready for filtering, aggregation, Parquet export, or joining with local tables.

### `ATTACH` to an ADBC Database

For persistent connections, users run `ATTACH` and then query the database as if local. The extension supports catalog lookups plus SELECT, INSERT, COPY, and CTAS statements:

```sql
ATTACH 'profile://mydb' AS mydb (TYPE adbc);
USE mydb.main;
SHOW ALL TABLES;
```

This reveals the `games` table with its eight columns and their types. A `SELECT * FROM games` returns the same five games. An INSERT adds Battleship (invented by Clifford Von Wickler in 1931 for ages 7+, 2 players, $12.99), bringing the total to six rows.

Users can then create local DuckDB tables from the attached database, or create new tables back in the ADBC-connected SQLite database, as demonstrated with a `game_inventors` table derived from the games data.

## Limitations

**Autocommit Only.** "The ADBC extension operates in autocommit mode," meaning queries take effect immediately and multi-statement transactions aren't yet supported.

**No Predicate or Projection Pushdown (for ATTACH).** When querying an attached table, the extension fetches all columns and rows, with DuckDB applying filters and projections locally. For large tables, the article recommends using `read_adbc` with a macro to push filtering down:

```sql
CREATE MACRO read_mydb(query) AS TABLE SELECT * FROM read_adbc('profile://mydb', query);
SELECT inventor FROM read_mydb('SELECT inventor FROM games WHERE name = ''Monopoly''');
```

Predicate and projection pushdown for attached tables is on the roadmap (GitHub issues #1 and #2).

## Key Features

**Streaming Bulk Ingest.** For INSERT or CTAS statements, rows are inserted in batches (roughly 2 million at a time by default) using ADBC's bulk ingest API. This approach keeps memory usage low even for datasets exceeding available RAM. The batch size can be adjusted via the `adbc_insert_buffer_size` setting.

**Connection Pooling.** After attaching an ADBC database, connections are reused across subsequent SQL statements instead of opening and closing new ones each time, reducing overhead. The pool size is tunable via `adbc_connection_pool_size`.

**Metadata Caching.** Schema and table metadata from attached databases are cached locally so that commands like `SHOW TABLES` don't require repeated remote queries. After a remote schema change, users run `CALL adbc_clear_cache()` to refresh the cache.

## Get Started

For existing ADBC users, this extension adds DuckDB to their client toolkit alongside driver managers, dataframe libraries, and other ADBC-speaking tools. Those new to ADBC can use the extension as an entry point for connecting DuckDB externally. The team invites feedback, bug reports, and contributions through:

- The [DuckDB ADBC extension documentation](https://duckdb.org/community%5Fextensions/extensions/adbc)
- The [extension's GitHub repo](https://github.com/columnar-tech/duckdb-adbc-client)
- Agent skills installable via `gh skill install columnar-tech/skills` or `npx skills add columnar-tech/skills`
