# Replace Athena with DuckDB (Lambda)

A CDK-deployed proof-of-concept that runs DuckDB inside AWS Lambda to query Parquet data on S3, achieving 75–86% cost reduction vs Athena for complex analytical queries. The key insight: Athena charges $5/TB scanned; Lambda charges per GB-second of compute. DuckDB's Parquet reader uses HTTP range requests to download only relevant columns, so Lambda execution time grows sub-linearly with data volume. For simple single-column scans, Lambda actually costs more — but those queries are rare in practice.

---

## Architecture

```
S3 (Parquet) ──HTTP range requests──▶ Lambda (Node.js 18)
                                        │
                                        ├── DuckDB Lambda Layer (native binary + httpfs + parquet extensions)
                                        ├── in-memory DuckDB (:memory:)
                                        └── CDK-deployed (NodejsFunction construct)
```

Three files do the work:

- **`lib/duckdb-sample-stack.ts`** — CDK stack: S3 bucket, Lambda function with 15-min timeout, DuckDB layer (region-specific ARN), IAM policy for S3 read. esbuild bundles TypeScript with `duckdb` externalized (it lives in the layer).
- **`src/duckdb-sample.ts`** — Lambda handler: creates in-memory DuckDB at module scope (warm-start persistence), receives SQL via `event.sql`, executes with `db.all()`, optionally generates date-range file lists for taxi data.
- **`bin/duckdb-sample.ts`** — CDK app entry point, deploys stack to us-east-2.

## Key techniques

**DuckDB httpfs + Parquet column pruning.** DuckDB's `read_parquet('s3://bucket/path')` makes HTTP range requests to S3, reading only the byte ranges for columns referenced in the query. A `SELECT vendor_id, count(*)` on a 20-column Parquet file downloads only the `vendor_id` column's data. Athena also does this — but charges for the full scan anyway.

**Lambda cost model inversion.** Athena pricing is linear in data scanned ($5/TB). Lambda pricing is linear in memory × time (GB-seconds). For complex queries, the Lambda compute cost grows slower than the Athena scan cost. Experimental results on NYC taxi data (6 months to 2 years):

| Query complexity | Lambda cost vs Athena |
|---|---|
| 1 column, GROUP BY | 100%+ (more expensive) |
| 2 columns, GROUP BY + AVG | 25–50% |
| 3 columns, GROUP BY + extract | <15% |
| 4 columns, GROUP BY + ORDER BY | <14% |

**512MB Lambda memory sweet spot.** Memory also controls CPU allocation. 128MB choked on S3 data transfer (30s+ for simple queries). 512MB gave the best cost/performance ratio. 3008MB added cost without proportional speedup — S3 transfer, not CPU, is the bottleneck.

**Lambda Layer for native binaries.** DuckDB's Node.js binding wraps a ~30MB native binary. The layer (from [serverless-duckdb](https://github.com/tobilg/serverless-duckdb)) bundles this with gLibc and extensions, keeping the function package small and deployable via CDK's `addLayers()`.

## Design decisions

- **File lists over Glue Catalog.** Instead of Athena's managed table metadata, the handler generates file paths programmatically from date ranges (`s3://bucket/2019/01/data.parquet`). Simpler, no catalog dependency, but no schema discovery or partition pruning metadata.
- **In-memory database at module scope.** `const db = new duckdb.Database(':memory:')` outside the handler means warm Lambda invocations reuse the instance. Cold starts pay the init cost once.
- **Region-specific layer ARNs.** The DuckDB layer is manually published per region; the CDK conditionally selects the right ARN. Only ap-northeast-1 and us-east-2 are supported in this demo.
- **No result serialization.** The handler returns `1` — it's a benchmark harness, not a query service. Production use would need result formatting, pagination, and an API Gateway.

## Limitations

- 10GB Lambda memory cap (experiments stayed under 126MB)
- 15-minute execution timeout (author suggests Fargate for longer queries)
- S3 transfer speed is the dominant cost factor, not query execution
- No managed service features: query history, workgroups, saved queries, Glue Catalog integration
- Cold start latency from Lambda Layer (not present in always-warm Athena)
- Only two regions supported for the DuckDB layer

## Comparison notes

[[Streambed]] also embeds DuckDB (for Iceberg queries via psql-wire) but targets CDC from Postgres rather than ad-hoc S3 analytics. [[AliSQL]] grafts DuckDB's columnar engine into MySQL — same insight that DuckDB's embedded engine is the right primitive. [[Shaper]] uses DuckDB for SQL-to-chart dashboards. All share the pattern: DuckDB as the zero-infrastructure analytical SQL engine. [[Drilldown Dashboards from a Single Parquet File]] takes that zero-infrastructure logic to its endpoint — Hyparquet, an 18KB browser Parquet reader, does the range scans client-side, so a customer dashboard runs with no Lambda, no DuckDB, no engine at all; the precomputed data-cube layout does the database work.

The cost-model inversion (compute pricing beats scan pricing for complex queries) applies to any scan-priced service: Redshift Spectrum, BigQuery (on-demand pricing), even Snowflake's credit model for certain workloads.

#tool #aws #duckdb #lambda #serverless #analytics #parquet #cost-optimization

---

_Source: [jiangzhuo/repalce-athena-with-duckdb](https://github.com/jiangzhuo/repalce-athena-with-duckdb), fetched 2026-06-21. Note: repo name typo ("repalce" for "replace"). Author's name is jiangzhuo._
