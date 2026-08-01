---
url: https://datafusion.apache.org/
title: Apache DataFusion — Extensible Query Engine
author: Apache Software Foundation
date_fetched: 2026-08-01
date_published: 2024
---

# Apache DataFusion

Apache DataFusion is an extensible query engine written in Rust that uses Apache Arrow as its in-memory format. The core DataFusion project contains libraries and binaries for developers building fast and feature-rich database and analytic systems, customized to particular workloads.

## Out-of-the-Box Capabilities

- **SQL and DataFrame APIs** — both are available by default.
- **Excellent performance** — benchmarks available from ClickHouse.
- **Built-in file format support** for CSV, Parquet, JSON, and Avro.
- **Extensive customization** options.
- **A full query planner**, plus a columnar, streaming, multi-threaded, vectorized execution engine, and partitioned data sources.

## Architecture

DataFusion is customizable at almost all points including additional data sources, query languages, functions, custom operators and more. Detailed architecture documentation is in the Contributor Guide.

## Related Subprojects

- **DataFusion Python** — Python interface for SQL and DataFrame queries.
- **DataFusion Java** — Java interface for SQL and DataFrame queries.
- **DataFusion Comet** — an accelerator for Apache Spark based on DataFusion.
- **DataFusion Ballista** — distributed processing extension for DataFusion enabling parallelized execution of workloads across multiple nodes.

## Documentation Structure

Three main guides:
1. **User Guide** — example usage, features, CLI, DataFrame API, SQL reference (data types, SELECT, subqueries, DDL, DML, EXPLAIN, information schema, operators, functions), configuration, explain plans, metrics, FAQ.
2. **Library User Guide** — upgrade guides (v46–55), extension APIs, SQL API, building logical plans, catalogs/schemas/tables, UDFs (scalar/window/aggregate/table), Spark-compatible functions, custom table providers, query optimizer.
3. **Contributor Guide** — communication, dev environment, architecture, testing, API health policy, release management, roadmap, governance, specifications.

## Project Status

Hosted under the Apache Software Foundation. Code on GitHub (github.com/apache/datafusion). Published on crates.io; API docs at docs.rs/datafusion. Has an official blog, code of conduct, and active contributor community.
