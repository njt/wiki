---
url: https://motherduck.com/docs/getting-started/data-warehouse/
date_fetched: 2026-10-09
---

# Data Warehousing Overview

## Introduction to MotherDuck for data warehousing

MotherDuck is a serverless cloud data warehouse built on DuckDB. Its hypertenancy architecture gives every user, service account, or agent a dedicated compute instance that starts in under a second and bills per second, so your whole team, humans and agents alike, gets sub-second answers without the legacy warehouse bill or cluster tuning. On top of DuckDB's fast analytical engine, MotherDuck adds cloud storage, sharing, and collaboration, and fits the tools data and analytics engineers already use: AI-assisted SQL, dbt for transformations, and hybrid local-cloud processing.

MotherDuck integrates with popular data tools including dltHub, Estuary, Fivetran, and Airbyte for data ingestion, dbt and dbt Cloud for transformations, Tableau, Power BI, and Looker for visualization, and Airflow and Dagster for orchestration. This enables teams to build data warehousing solutions using their existing tools.

## Data ingestion

A quick way to get data into MotherDuck is using ecosystem partners like dltHub, Estuary, Fivetran, and Airbyte. For scheduled ingest without external infrastructure, Flights run Python pipelines natively on MotherDuck: see the guides on ingesting S3 parquet files on a schedule and running a dlt ingest pipeline from a Flight.

MotherDuck is very flexible with how to load your data:

- **From data you have on your filesystem:**If you have CSVs, JSON files, or DuckDB databases sitting around, you can load them straight into your MotherDuck data warehouse.
- **From a lakehouse on a cloud object store:**MotherDuck works with open table formats. Create and query DuckLake tables backed by your own object storage, or attach an Iceberg REST catalog such as Databricks-managed Iceberg or Cloudflare R2 Data Catalog to read from and write back to Iceberg tables. DuckDB's abstractions for Secrets, Object Storage, and file formats also let you query raw parquet, CSV, and JSON files from object storage with only SQL. Though not as performant as MotherDuck's native storage layer, this lets you query your infrequently-accessed data directly from your data lake.
- **Using Native APIs in many languages:**DuckDB supports numerous languages such as C++, Python, and Java, in addition to its own mostly Postgres-compatible SQL dialect. Using these languages, Data Engineers and Developers can integrate with MotherDuck without having to pick up yet-another-language.

### Best practices for programmatic loading

The fastest way to load data is to load single tables in large batches, saturating the network connection between MotherDuck and the source data. DuckDB is incredibly good at handling both files and some kinds of in-memory objects, like Arrow dataframes. As an aside, Parquet files compress at 5-10x compared to CSV, which means you can get 5-10x more throughput by using Parquet files. Similarly, open table formats like Delta & Iceberg share those performance gains.

On the other hand, small writes on multiple tables will lead to suboptimal performance. While MotherDuck does indeed offer ACID compliance, it is not an OLTP system like Postgres! Significantly better performance can be achieved by using queues to batch writes to tables. While some latency is introduced with this methodology, the improvement in throughput should far outweigh the cost of doing small writes.

Streaming workloads are better suited to be handled with queues in front of MotherDuck.

## Transforming data

Once data is loaded into MotherDuck, it must be transformed into a model that matches the business purpose and needs. This can be done directly in MotherDuck using the powerful library of SQL functions offered by DuckDB. Many data engineers prefer to use data transformation tools like the open source dbt Core. You can connect dbt Cloud to MotherDuck through the Postgres endpoint using dbt's built-in Postgres adapter, or run dbt transformations from a Flight to schedule them inside MotherDuck. More details specifically about using dbt with MotherDuck can be read in the blog on this topic.

For more in-depth reading, the free **DuckDB in Action eBook** explores these concepts with real-world examples.

## Sharing data

Once your data is loaded into MotherDuck and appropriately transformed for use by your analysts, you can make that data available using MotherDuck's sharing capabilities. Grant the Share to the Explorer role to make it available to every preset role in your organization. Queries run by Explorers use isolated compute and do not affect the data pipelines managed by Builders or Admins.

## Serving data analytics

Do you want to serve reports or dashboards for your users? Connect popular BI tools like Power BI, Tableau Cloud, and Looker through the Postgres endpoint, which works with any client that speaks the PostgreSQL wire protocol.

For internal dashboards, Dives let you build interactive visualizations from natural language with an AI agent and share them with your team, no separate BI tool required.

Agents are data consumers too. Connect AI assistants like Claude or Cursor through the MotherDuck MCP Server, and use Guides, markdown documents with your metric definitions and query conventions, to keep agent answers accurate and consistent.

### Ducks all the way down: building data apps

Because DuckDB is an extremely efficient SQL engine inside a ~20MB executable, you can also run it inside the web browser through WASM to build highly interactive data apps and visualizations with near-zero latency.

## Scaling up & out for DWH use cases

Furthermore, MotherDuck has a unique scaling model, of which there are four key concepts relevant for Data Warehousing.

### Vertical scaling

Compute can scale up with larger DuckDB compute instances called Ducklings. MotherDuck offers 5 sizes: Pulse, Standard, Jumbo, Mega, and Giga.

Unlike other data warehouses, every Duckling (compute instance) is isolated from each other: one user's queries will not impact another user's from completing. This hypertenancy model assures you can size your warehouse correctly and use your resources very efficiently.

To keep compute costs in check, configure the Duckling cooldown period per user or service account, and end batch pipelines with `SHUTDOWN` to stop the Duckling without waiting for the cooldown.

### Horizontal scaling

For serving data to BI tools or other spiky consumers, Read Scaling Replicas can absorb the loads and maintain low latency on user interactivity. These should be owned by the same user or service accounts that run production jobs, although they can also leverage `SHARES` depending on preferences.

### Hypertenancy

Especially for production runs, use separate user accounts or service accounts with dedicated compute for updating and maintaining core tables.

### Distributed DuckDB

DuckDB and MotherDuck work together as a distributed system that automatically optimizes query execution between local and cloud resources through Dual Execution, enabling efficient data access regardless of location.

## Orchestration

To keep data up to date inside MotherDuck, start with Flights: scheduled Python jobs that run natively on MotherDuck, so you can load and transform data on a cron schedule without external infrastructure. You manage Flights through SQL, the UI, or the MCP Server, which means an AI agent can build and maintain them for you.

For more complex data engineering pipelines that span multiple systems, an external orchestrator like Airflow or Dagster runs jobs in specific orders to load and transform data, as well as managing workflow and observability.

If this is your first data warehouse, you might consider starting with GitHub actions or cron jobs to orchestrate your data pipelines.

For a more in-depth guide, check out the Data Warehousing Guide

### Try MotherDuck for free

Sign up and start building your data warehouse in minutes. No credit card required.
