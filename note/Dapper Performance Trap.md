# Dapper Performance Trap

When Dapper sends a C# `string` to a SQL Server `varchar` column, ADO.NET defaults to `nvarchar(4000)`. SQL Server then implicitly converts every row in the column before comparing — defeating indexes and causing full table scans. Kevin Griffin found this in production where a single Dapper query was consuming a significant chunk of total database CPU. His benchmarks: **176x slower** on 1M rows, **268x** on 500K. The fix: `DbType.AnsiString` instead of the default. Two characters that are completely invisible in C#.

---

## The Discovery

Griffin's production system was running hot — CPU averaging over 50% and spiking into the 90s. The top query by CPU time was a straightforward Dapper call. Simple WHERE clause on an indexed column. Should have been microseconds. Instead, averaging thousands of milliseconds of CPU per execution, hundreds of thousands of times per day.

He stared at it for way too long before figuring out what was happening. The query looked perfect. The index was correct. But the parameter type was wrong.

## Why It Happens

```csharp
// This is the silent killer:
var result = await connection.QueryFirstOrDefaultAsync<Product>(
    sql, new { productCode });
```

ADO.NET maps `System.String` → `nvarchar(4000)`. Always. It's a reasonable default — `nvarchar` handles Unicode, `4000` covers most cases. But if your column is `varchar`, SQL Server's type precedence rules convert the column (VARCHAR) to match the parameter (NVARCHAR), not the other way around. Per row.

This produces `CONVERT_IMPLICIT` in the execution plan:
```
CONVERT_IMPLICIT(nvarchar(255), [Sales].[ProductCode], 0)
```

SQL Server is saying: "I had a perfectly good index, but you made me convert every row to compare against your Unicode parameter, so I couldn't use it."

## Collation Details

Severity depends on collation. `SQL_Latin1_General_CP1_CI_AS` (the most common default) gives full index scans — worst case. Some Windows collations like `Latin1_General_CI_AS` may still allow seeks, but the conversion overhead remains. Match your types regardless.

## The Numbers

The article originally didn't have benchmarks. It hit Hacker News, the commenters demanded numbers, and Griffin "vibe coded a BenchmarkDotNet suite" with 1M products and 500K orders. 1,000 single-row lookups per scenario:

| Scenario | nvarchar (default) | varchar (fixed) | Slower |
|---|---|---|---|
| ProductCode (1M rows) | 23 sec | 0.13 sec | **176x** |
| OrderNumber (500K rows) | 35 sec | 0.13 sec | **268x** |

Execution plans:

| | nvarchar (default) | varchar (correct) |
|---|---|---|
| Operator | Index Scan | Index Seek |
| CONVERT_IMPLICIT | Yes | No |
| Est. Query Cost | 7.78 | 0.007 |

Over 1,000x more expensive by SQL Server's own cost estimate. No schema changes, no new indexes, no query rewrites. Just the correct parameter type.

## The Fix

```csharp
var parameters = new DynamicParameters();
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql, parameters);
```

`DbType.AnsiString` → `varchar`. `DbType.String` (the default) → `nvarchar`. Match the `size:` to your column definition — it helps SQL Server reuse cached query plans.

Inline alternative using `DbString`:
```csharp
new { productCode = new DbString { Value = productCode, IsAnsi = true, Length = 100 } }
```

## Protect Your Fix

Griffin's strongest advice: **comment why you're using DynamicParameters**. Without documentation, a future refactor (human or agent) will "simplify" it back to `new { productCode }` and silently reintroduce the problem. He writes:

> "The verbosity is the point. It's a speed bump that prevents someone from accidentally undoing a critical performance fix."

```csharp
var parameters = new DynamicParameters();
// DbType.AnsiString required: Products.ProductCode is varchar(100). Without it,
// Dapper sends nvarchar(4000) which causes CONVERT_IMPLICIT and defeats index seeks.
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
```

## How to Find This In Your Code

1. **Query Store:** Look for `nvarchar(4000)` parameters in high-CPU queries (full SQL above in [[summary/dapper-performance-trap]]).
2. **Execution plans:** Find `CONVERT_IMPLICIT` warnings on queries filtering `varchar` columns.
3. **Code search:** Grep for `new {` near Dapper calls targeting `varchar` tables.

## Key Quotes

> "A two-character type mismatch that was completely invisible in the C# code."

> "I stared at the query for way too long before I figured out what was happening."

> "Everything works — it just works slowly, and you don't know why until you dig into execution plans or query store data."

> "Every anonymous object passing a string to a varchar column is a potential full table scan hiding in plain sight."

## Critical Analysis

This is a well-executed diagnostic: here's the problem, here's the benchmark proving it, here's the fix. The 176x number makes the case conclusively. Griffin's meta-note about AI collaboration is refreshingly candid — he discloses Claude helped research, structure, and benchmark the article. The HN-driven benchmark addition is the piece that elevates this from good to definitive: without the numbers, it's a war story; with them, it's a production postmortem anyone can cite.

The missing conversation: this same class of bug exists in Entity Framework, Hibernate, and every other ORM that abstracts SQL types. The article is Dapper-specific, but the principle is universal — type systems are contracts, and violating them silently (as ADO.NET does) creates bugs that only surface under load.

This is particularly relevant to AI-generated code. An LLM producing Dapper queries will generate the `nvarchar` default every time, because every example in its training data does. See [[dotnet Slopwatch]] for a tool that catches vibe-coding warts like this.

The same diagnostic pattern applies to pagination: [[SQL Pagination — Offset vs Seek Method]] shows that `OFFSET` pagination also silently defeats indexes — the query produces correct results at the cost of scanning every preceding row — and the fix (seek method / keyset pagination) similarly requires understanding what the execution plan is actually doing, not just whether the results look right.

## Rule

Column is `varchar` → `DbType.AnsiString`. Column is `nvarchar` → default `DbType.String` is fine. Match sizes. Comment your DynamicParameters. Audit your queries today.

---
*Sources: [[summary/dapper-performance-trap]]*
*Last updated: 2026-05-18*
