---
url: https://github.com/jiangzhuo/repalce-athena-with-duckdb
title: A Cheap Alternative to AWS Athena — Lambda × DuckDB
author: jiangzhuo
date_fetched: 2026-06-21
date_published: 2023
topics:
  - misc
---

# A Cheap Alternative to AWS Athena: Lambda × DuckDB

GitHub repo by jiangzhuo demonstrating a CDK-deployed alternative to AWS Athena: DuckDB running inside AWS Lambda queries Parquet data directly from S3, trading Athena's per-TB-scan pricing ($5/TB) for Lambda's per-GB-second compute pricing.

## Repository structure

```
bin/duckdb-sample.ts          — CDK app entry point (deploys to us-east-2)
lib/duckdb-sample-stack.ts    — CDK stack: S3 bucket, Lambda, IAM, DuckDB layer
src/duckdb-sample.ts          — Lambda handler (DuckDB in-memory, S3 Parquet queries)
test/duckdb-sample.test.ts    — Placeholder test (commented out)
data/sql{1-4}.json            — Test event payloads for Lambda invocation
data/*.parquet                — Sample TPC-H data (customer, lineitem, orders, nation)
cdk.json, tsconfig.json       — CDK and TypeScript configuration
```

## How it works

1. **CDK deploys a Node.js 18 Lambda** with the DuckDB Node.js binding (`duckdb` npm package v0.7.1)
2. **DuckDB native binary is bundled as a Lambda Layer** (via [serverless-duckdb](https://github.com/tobilg/serverless-duckdb)), keeping the function deployment package small. The layer includes gLibc and essential extensions: `httpfs` (S3/HTTP access) and `parquet`
3. **Lambda receives SQL queries via event payload** (`event.sql`), creates an in-memory DuckDB instance (`new duckdb.Database(':memory:')`), and executes them using `read_parquet('s3://bucket/path')` — DuckDB streams Parquet files directly from S3 via HTTP range requests
4. **esbuild bundles the TypeScript**, configured to externalize `duckdb` since it lives in the layer

## The secret sauce: DuckDB's httpfs extension

DuckDB's `httpfs` extension makes HTTP range requests to S3 objects, reading only the byte ranges needed for the query. Because Parquet is columnar, a query touching only 2 columns out of 20 only downloads those 2 columns' data. This is how DuckDB-on-Lambda avoids the full-data-scan cost model of Athena.

Athena also reads Parquet efficiently, but charges $5/TB *scanned* regardless of efficiency. Lambda charges per GB-second of compute time — a fundamentally different cost model that favors complex queries over simple ones.

## Experimental results (NYC taxi data)

Tested 4 SQL queries of increasing complexity against 6 months, 1 year, and 2 years of NYC taxi Parquet data, with Lambda memory configurations from 128MB to 3008MB:

- **SQL1** (single-column `GROUP BY`): Lambda was *more expensive* than Athena. Trivial query, scan cost negligible, Lambda overhead dominated.
- **SQL2** (two-column `GROUP BY` with `AVG`): Lambda cost dropped to **25–50% of Athena** at 512MB.
- **SQL3** (three-column `GROUP BY` with `extract`): Lambda cost fell to **<15% of Athena** at 512MB.
- **SQL4** (four-column `GROUP BY` with ordering): Lambda cost was **<14% of Athena** at 512MB.

The pattern: **the more complex the query, the greater the Lambda advantage**. Athena's per-TB scanning cost grows linearly with data volume; Lambda's execution time grows sub-linearly because DuckDB reads only relevant byte ranges from Parquet.

The 512MB memory configuration was the cost sweet spot. 128MB choked on data transfer, 3008MB added cost without proportional speedup.

## Core Lambda code

The Lambda handler is deceptively simple — about 40 lines of actual logic:

```typescript
import * as duckdb from 'duckdb';
import * as util from "util";
import { Handler } from 'aws-lambda';

const db = new duckdb.Database(':memory:');

export const handler: Handler = async (event) => {
  const sql = event.sql;
  const dbAllPromise = util.promisify(db.all.bind(db));
  async function executeQuery(query: string) {
    console.time(query);
    const result = await dbAllPromise(query);
    console.timeEnd(query);
    return result;
  }
  const results = await Promise.all([sql].map(executeQuery));
  return 1;
}
```

The in-memory database is created at module load time (outside the handler), so it persists across warm Lambda invocations.

For the taxi data experiments, the handler builds file lists dynamically from date ranges, generating paths like `s3://ursa-labs-taxi-data/2019/01/data.parquet` and passing them to `read_parquet([...])`.

## CDK infrastructure

The stack (`lib/duckdb-sample-stack.ts`):
- Creates an S3 bucket for sample data
- Deploys TPC-H Parquet files via `BucketDeployment`
- Creates a `NodejsFunction` (Lambda) with:
  - Node.js 18 runtime, x86_64 architecture
  - 128MB default memory, 15-minute timeout
  - esbuild bundling with `externalModules: ['duckdb']`
  - Region-specific DuckDB Lambda Layer ARNs (ap-northeast-1, us-east-2)
  - S3 read grants + explicit `s3:*` policy for the public taxi data bucket

## Key techniques

1. **In-memory DuckDB** — `':memory:'` database, no persistent storage, stateless between cold starts
2. **Lambda Layer for native deps** — DuckDB's native binary (~30MB) lives in a layer, separate from function code. esbuild excludes it from bundling
3. **`read_parquet()` with S3 paths** — DuckDB's Parquet reader natively understands `s3://` URIs via httpfs
4. **Parquet column pruning via HTTP range requests** — DuckDB only fetches the byte ranges for relevant columns, not the full file
5. **CDK `NodejsFunction` construct** — Auto-bundles TypeScript, auto-creates IAM role, handles deployment
6. **`EXPLAIN ANALYZE` queries** — The test harness wraps all queries in `EXPLAIN ANALYZE` for performance measurement

## Design decisions

- **Region-specific layer ARNs** — The DuckDB Lambda Layer is manually built per region; the CDK conditionally selects the right ARN. Only two regions supported (ap-northeast-1, us-east-2)
- **S3 wildcard IAM policy** (`s3:*` on the taxi data bucket) — overly broad for production but simple for a demo
- **File list generation in Lambda** — Instead of Athena's Glue Catalog partitioning, the handler generates file paths programmatically from date ranges. No metadata catalog needed
- **Memory over CPU optimization** — Lambda memory also scales CPU; the 512MB sweet spot reflects that S3 transfer (I/O) is the bottleneck, not query computation
- **No query result handling** — The handler returns `1`, not query results. This is a benchmark harness, not a production service

## Limitations and trade-offs

1. **10GB Lambda memory ceiling** — but experiments used <126MB even for multi-year datasets
2. **15-minute Lambda timeout** — author suggests AWS Fargate for longer queries
3. **S3 transfer is the bottleneck**, not query execution — httpfs performance dominates. Suggested mitigation: pre-load data into EFS to eliminate per-query S3 transfers
4. **No Glue Data Catalog integration** — file paths must be manually constructed or enumerated. No schema discovery
5. **No query result API** — the benchmark handler returns `1`. A production version would need result serialization, pagination, and an API Gateway frontend
6. **Single-region deployment** — DuckDB layer only available in two regions
7. **Cold start latency** — the Lambda Layer adds startup time not present in Athena's always-warm model

## Comparison notes

- **vs Athena**: Cheaper for complex analytical queries (75-86% savings), more expensive for trivial single-column scans. No managed service niceties (Glue Catalog, query history, workgroups). Self-managed infrastructure via CDK.
- **vs Streambed**: [[Streambed]] also embeds DuckDB (for querying Iceberg tables via psql-wire), but targets CDC from Postgres rather than ad-hoc S3 analytics. Complementary: Streambed for continuous replication, this for on-demand querying.
- **vs AliSQL**: [[AliSQL]] grafts DuckDB's columnar engine into MySQL for hybrid OLTP/OLAP. This project runs DuckDB standalone. Both share the insight that DuckDB's embedded columnar engine is the right primitive for analytical workloads.
- **vs Shaper**: [[Shaper]] uses DuckDB behind a SQL-to-chart dashboard. Both leverage DuckDB's zero-infrastructure query model, but this project is Lambda-hosted while Shaper is likely local/CLI.
- **vs Redshift Spectrum / Athena**: Both AWS services charge per-TB-scanned. This project's insight — that Lambda's compute pricing can beat scan-based pricing for complex queries — applies to any scan-priced query service.

## Tags

#tool #database #aws #duckdb #lambda #serverless #analytics #parquet #cost-optimization #cdk #typescript

## See also

- [[Streambed]] — Postgres-to-Iceberg CDC with embedded DuckDB query server
- [[AliSQL]] — DuckDB columnar engine grafted into MySQL
- [[Shaper]] — SQL-driven dashboards with DuckDB
- [[Databases and Data]] — hub page
- [[floci]] — Free local AWS emulator (related AWS tooling)

---

_Source: [jiangzhuo/repalce-athena-with-duckdb](https://github.com/jiangzhuo/repalce-athena-with-duckdb), fetched 2026-06-21. Note: repo name has a typo ("repalce" for "replace")._
