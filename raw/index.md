---
url: https://clickhouse.com/docs/chdb/index
date_fetched: 2026-10-18
---

## Key features

- **In-process SQL OLAP Engine**- Powered by ClickHouse, no need to install ClickHouse server
- **Multiple data formats**- Input & Output support for Parquet, CSV, JSON, Arrow, ORC and 70+ more formats
- **Minimized data copy**- From C++ to Python with python memoryview
- **Rich Python Ecosystem Integration**- Native support for Pandas, Arrow, DB API 2.0, seamlessly fits into existing data science workflows
- **Zero dependencies**- No need for external database installations
- **DataStore API**- Pandas-compatible API with SQL optimization, supporting 630+ methods

## DataStore: Pandas-Compatible API

**NEW!**DataStore provides a pandas-compatible API that combines familiar pandas syntax with ClickHouse performance.

### One-Line Migration

### Performance Highlights

*Benchmarks on 10M rows*

### DataStore Features

- **630+ API methods**- 209 pandas DataFrame methods, 185+ accessor methods
- **Lazy evaluation**- Operations compile to optimized SQL
- **SQL pushdown**- Filters and aggregations run at the data source
- **Universal data sources**- Read from files, S3, databases, data lakes

## What languages are supported by chDB?

chDB has the following language bindings:## How do I get started?

- If you’re using Go, Rust, NodeJS, Bun or C and C++, take a look at the corresponding language pages.
- If you’re using Python, see the getting started developer guide or the chDB on-demand course.

### For pandas Users

Start with the DataStore API for a familiar pandas experience with ClickHouse performance:- DataStore Quickstart - Installation and one-line migration
- Migration from pandas - Step-by-step migration guide
- Pandas Cookbook - Common patterns
- Key Differences - Important differences from pandas
- Performance Guide - Optimization tips

### DataStore API Reference

- Factory Methods - Create from files, databases, cloud storage
- Query Building - SQL-style operations
- Pandas Compatibility - 209 compatible methods
- Accessors - .str, .dt, .arr, .json, .url, .ip, .geo
- Configuration - Engine, logging, profiling
- Debugging - explain(), profiling, logging

### SQL API Guides

- Python API Reference - Complete SQL API documentation
- JupySQL
- Querying Pandas
- Querying Apache Arrow
- Querying data in S3
- Querying Parquet files
- Querying remote ClickHouse
- Using clickhouse-local database

## An introductory video

Watch a brief introduction to chDB and learn how it brings ClickHouse’s power to your Python environment:## Performance benchmarks

chDB delivers exceptional performance across different scenarios:- **ClickBench of embedded engines**- SQL API performance comparison
- **DataFrame Benchmark**- DataFrame engines comparison
- **DataStore vs Pandas**- Up to 20x faster than pandas on common operations

## About chDB

- Read the full story about the birth of the chDB project on blog
- Read about chDB and its use cases on the Blog
- Take the chDB on-demand course
- Discover chDB in your browser using codapi examples
- More examples see (https://github.com/chdb-io/chdb/tree/main/examples)
