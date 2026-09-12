---
title: "How C# Strings Silently Kill Your SQL Server Indexes in Dapper"
url: https://consultwithgriff.com/dapper-nvarchar-implicit-conversion-performance-trap
author: Kevin Griffin
date_fetched: 2026-05-18
date_published: 2025-05-01
section: "C# and .NET"
topics:
  - software-engineering-craft
  - databases-and-data
---

# How C# Strings Silently Kill Your SQL Server Indexes in Dapper

Author's note: "My good friend Claude (yes, the AI) helped me write this one. The underlying issue is 100% real — we found it in a production system using AI-assisted code analysis. Claude helped me research the internals, structure the explanation, and build the benchmark suite."

## The Discovery

Production system running hot — CPU averaging over 50% and spiking into the 90s. Top query by CPU time was a straightforward Dapper query with a simple WHERE clause on an indexed column. Should have been lightning fast. Instead, averaging thousands of milliseconds of CPU per execution across hundreds of thousands of executions per day.

## The Mechanism

When you pass a C# `string` through a Dapper anonymous object:

```csharp
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql, new { productCode });
```

ADO.NET maps `System.String` to `nvarchar(4000)` by default. If the target column is `varchar`, SQL Server converts every value in the column to `nvarchar` before comparing. This is `CONVERT_IMPLICIT`, and it means SQL Server can't use the index. Full scan every time.

The execution plan shows:
```
CONVERT_IMPLICIT(nvarchar(255), [Sales].[ProductCode], 0)
```

## Collation Matters

With `SQL_Latin1_General_CP1_CI_AS` (the most common default), you get full index scans — worst case. Some Windows collations (like `Latin1_General_CI_AS`) may still allow index seeks, but the implicit conversion overhead remains. Either way, matching parameter types is the right call.

## Benchmark Results

The original article didn't have benchmarks. After it hit Hacker News, commenters demanded them. The author "vibe coded a BenchmarkDotNet suite" with 1M products and 500K orders, running 1,000 single-row lookups per scenario:

| Scenario | nvarchar (default) | varchar (fixed) | How Much Slower |
|---|---|---|---|
| ProductCode (1M rows) | 23 sec | 0.13 sec | 176x |
| OrderNumber (500K rows) | 35 sec | 0.13 sec | 268x |

Execution plan comparison:

| Metric | nvarchar (Dapper default) | varchar (correct) |
|---|---|---|
| Plan Operator | Index Scan | Index Seek |
| CONVERT_IMPLICIT | Yes | No |
| Estimated Query Cost | 7.78 | 0.007 |

SQL Server's own cost estimate: over 1,000x more expensive with the wrong type.

## The Fix

```csharp
var parameters = new DynamicParameters();
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql, parameters);
```

`DbType.AnsiString` → `varchar`. `DbType.String` (default) → `nvarchar`. Match the size to the column definition.

`DbString` alternative for inline definitions:

```csharp
new { productCode = new DbString { Value = productCode, IsAnsi = true, Length = 100 } }
```

## Protect Your Fix

Comment why you're using `DynamicParameters` instead of anonymous objects. Without a comment, a well-meaning developer will "simplify" it back during a future refactor and reintroduce the problem.

```csharp
var parameters = new DynamicParameters();
// DbType.AnsiString required: Products.ProductCode is varchar(100). Without it,
// Dapper sends nvarchar(4000) which causes CONVERT_IMPLICIT on every row and
// defeats index seeks.
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
```

## Detection Queries

**Query Store check:**
```sql
SELECT TOP 20 qsqt.query_sql_text, qsrs.avg_cpu_time, qsrs.count_executions
FROM sys.query_store_runtime_stats qsrs
JOIN sys.query_store_plan qsp ON qsrs.plan_id = qsp.plan_id
JOIN sys.query_store_query qsq ON qsp.query_id = qsq.query_id
JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
WHERE qsqt.query_sql_text LIKE '%@%nvarchar(4000)%'
ORDER BY qsrs.avg_cpu_time * qsrs.count_executions DESC;
```

**Execution plan inspection:** Look for `CONVERT_IMPLICIT` warnings on queries filtering `varchar` columns.

**Code search:** Find Dapper calls passing string parameters through anonymous objects to queries on varchar columns: `await connection.QueryAsync<T>(sql, new { someVarcharColumn });`

## Rule of Thumb

If the column is `varchar`, use `DbType.AnsiString`. If the column is `nvarchar`, the default `DbType.String` is fine. Match the parameter type to the column type, and match the size to the column size.

## Key Quotes

"A two-character type mismatch that was completely invisible in the C# code."

"I stared at the query for way too long before I figured out what was happening."

"Everything works — it just works slowly, and you don't know why until you dig into execution plans or query store data."

"Every anonymous object passing a string to a varchar column is a potential full table scan hiding in plain sight."

## About the Author

Kevin Griffin has been running production .NET applications and teaching developers for over two decades. 16-time Microsoft MVP specializing in ASP.NET Core and Azure. Runs his own SaaS products alongside consulting at Swift Kick. Hosted the Hampton Roads .NET User Group since 2009 and founded RevolutionVA, the nonprofit behind Hampton Roads DevFest.
