---
url: https://consultwithgriff.com/dapper-nvarchar-implicit-conversion-performance-trap
date_fetched: 2026-07-05
backfilled: true
---

# How C# Strings Silently Kill Your SQL Server Indexes in Dapper


A note on how this article came to be:My good friend Claude (yes, the AI) helped me write this one. The underlying issue is 100% real — we found it in a production system using AI-assisted code analysis. Claude helped me research the internals, structure the explanation, and build the benchmark suite. I’m not going to pretend otherwise. The future of writing is collaborative, folks.

I recently spent time digging into a production performance issue. The application was running hot — CPU averaging over 50% and spiking into the 90s. We pulled a diagnostic snapshot and started working through the top queries by CPU time.

The number one offender? A straightforward Dapper query. Simple WHERE clause on an indexed column. Should have been lightning fast. Instead, it was averaging thousands of milliseconds of CPU per execution across hundreds of thousands of executions per day.

A two-character type mismatch that was completely invisible in the C# code. I stared at the query for way too long before I figured out what was happening.

## So What’s Actually Happening?

Here’s a pattern you’ll find in almost every .NET project that uses Dapper:

```
const string sql = "SELECT * FROM Products WHERE ProductCode = @productCode";
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql, new { productCode });
```
Clean. Simple. And if `ProductCode` is a `varchar` column in your database, it’s silently destroying your query performance.

When you pass a C# `string` through an anonymous object, Dapper maps it to `nvarchar(4000)`. That’s the default mapping for `System.String` in ADO.NET — and honestly, it makes sense from a “safe default” perspective. But if your column is `varchar`, SQL Server has to convert *every single value in the column* to `nvarchar` before it can compare. This is called **CONVERT_IMPLICIT**, and it means SQL Server can’t use your index. Full scan. Every time.

You can see it hiding in your execution plans:

`CONVERT_IMPLICIT(nvarchar(255), [Sales].[ProductCode], 0)`That’s SQL Server telling you: “I had a perfectly good index, but you made me convert every row to compare against your Unicode parameter, so I couldn’t use it.”

## Ok, But How Bad Can It Really Be?

The math on this is brutal. Let’s say you have a table with a million rows and a nonclustered index on `ProductCode`. With correct parameter types, SQL Server performs an **index seek** — it jumps directly to the matching row. A handful of logical reads. Microseconds.

With the implicit conversion, SQL Server performs an **index scan** — it reads every single row in the index, converts each value, and then compares. Instead of a handful of logical reads, you’re doing tens of thousands. Multiply that by the number of times the query executes per day, and you’ve got a serious CPU problem.

In our case, a single query was responsible for a significant chunk of the database server’s total CPU consumption. Not because the query was complex. Not because the table was poorly indexed. Just because of a type mismatch in the parameter.


A note on collation:The severity here depends on your database collation. With the most common default (`SQL_Latin1_General_CP1_CI_AS`), you get full index scans — the worst case. Some Windows collations (like`Latin1_General_CI_AS`) may still allow index seeks, but the implicit conversion overhead remains. Either way, matching your parameter types is the right call.

## The Fix

The fix is almost embarrassingly simple. You just tell Dapper explicitly that the parameter is `varchar`, not `nvarchar`. You do this with `DynamicParameters` and `DbType.AnsiString`:

```
const string sql = "SELECT * FROM Products WHERE ProductCode = @productCode";
var parameters = new DynamicParameters();
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql, parameters);
```
And that’s it. `DbType.AnsiString` tells ADO.NET to send a `varchar` parameter. `DbType.String` (the default for C# `string`) sends `nvarchar`.

The `size` parameter should match your column definition. If your column is `varchar(255)`, use `size: 255`. This helps SQL Server match the parameter type exactly to the column type and reuse cached query plans efficiently.

Dapper also provides `DbString` as a more concise alternative if you prefer staying with anonymous objects:

```
var result = await connection.QueryFirstOrDefaultAsync<Product>(sql,
    new { productCode = new DbString { Value = productCode, IsAnsi = true, Length = 100 } });
```
Both approaches produce the same result — a `varchar` parameter instead of `nvarchar`.

## Before and After

The difference is immediate and dramatic:

| Metric | Before (nvarchar) | After (varchar) | 
|---|---|---|
| Scan type | Index SCAN | Index SEEK | 
| Logical reads | Tens of thousands | Single digits | 
| CPU per execution | Milliseconds | Microseconds | 

No schema changes. No new indexes. No query rewrites. Just telling Dapper the correct parameter type.

## Benchmark Results

When this article hit Hacker News, the folks there (bless their hearts) wanted benchmarks. Fair enough — I’d want them too.

So I vibe coded a BenchmarkDotNet suite with 1 million products and 500,000 orders seeded into SQL Server, then ran 1,000 single-row lookups for each scenario. Here’s what happened:

| Scenario | nvarchar (default) | varchar (fixed) | How Much Slower | 
|---|---|---|---|
| ProductCode (1M rows) | 23 sec | 0.13 sec | 176x | 
| OrderNumber (500K rows) | 35 sec | 0.13 sec | 268x | 

Not a typo. **176x slower** on the product table. 268x on orders.

The execution plans tell the whole story:

| Metric | nvarchar (Dapper default) | varchar (correct) | 
|---|---|---|
| Plan Operator | Index Scan | Index Seek | 
| CONVERT_IMPLICIT | Yes | No | 
| Estimated Query Cost | 7.78 | 0.007 | 

That’s **over 1,000x more expensive** by SQL Server’s own cost estimate. On a million-row table, you’re reading the entire index instead of jumping straight to the row you need. Ask me how I know.

Want to see it yourself? Clone the benchmark repo, point it at a local SQL Server instance, and run `dotnet run -c Release -- --setup` to seed the data. Then `dotnet run -c Release -- --walltime` to watch the numbers roll in.

## How to Find This in Your Application

If you suspect you have this problem, there are a few ways to find it:

**1. Check Query Store for implicit conversions:**

```
SELECT TOP 20
    qsqt.query_sql_text,
    qsrs.avg_cpu_time,
    qsrs.count_executions
FROM sys.query_store_runtime_stats qsrs
JOIN sys.query_store_plan qsp ON qsrs.plan_id = qsp.plan_id
JOIN sys.query_store_query qsq ON qsp.query_id = qsq.query_id
JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
WHERE qsqt.query_sql_text LIKE '%@%nvarchar(4000)%'
ORDER BY qsrs.avg_cpu_time * qsrs.count_executions DESC;
```
**2. Look for  CONVERT_IMPLICIT in execution plans:**

If you see `CONVERT_IMPLICIT` warnings in your actual execution plan for a query that filters on a `varchar` column, you’ve found it.

**3. Search your C# code:**

Look for Dapper calls that pass string parameters through anonymous objects to queries on `varchar` columns:

```
// This is the pattern that causes the problem
await connection.QueryAsync<T>(sql, new { someVarcharColumn });
```
## The Rule of Thumb

Simple: if the column is `varchar`, use `DbType.AnsiString`. If the column is `nvarchar`, the default `DbType.String` is fine. Match the parameter type to the column type, and match the size to the column size.

## Protect Your Fix With Comments

One thing I’d strongly recommend: comment *why* you’re using `DynamicParameters` instead of anonymous objects. Because I guarantee you — without a comment, some well-meaning developer will “simplify” it back to `new { productCode }` during a future refactor and reintroduce the problem. (Ask me how I know.)

```
var parameters = new DynamicParameters();
// DbType.AnsiString required: Products.ProductCode is varchar(100). Without it, Dapper sends
// nvarchar(4000) which causes CONVERT_IMPLICIT on every row and defeats index seeks.
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
```
The verbosity is the point. It’s a speed bump that prevents someone from accidentally undoing a critical performance fix.

## Go Audit Your Queries

This is one of those bugs that’s nearly invisible. The code looks correct. The query returns the right results. There are no errors in the logs. Everything *works* — it just works slowly, and you don’t know why until you dig into execution plans or query store data.

If you’re using Dapper with SQL Server and your columns are `varchar`, go audit your parameter usage. Seriously, do it today. Every anonymous object passing a string to a `varchar` column is a potential full table scan hiding in plain sight.

If you’re newer to Dapper, check out my introduction to what Dapper is and why you should consider it for your .NET projects. And if you’re building paginated queries, you might also like my article on using COUNT(*) OVER() to get pagination counts in a single query.

Have you run into this one before? I’d love to hear your war stories — hit me up on X, Bluesky, or LinkedIn.

## Further Reading

- DbType Enumeration — Microsoft’s docs on ADO.NET data types, including the distinction between `AnsiString`and`String`.
- Query Processing Architecture Guide — Deep dive into how SQL Server processes queries, including implicit conversions and plan caching.
- Data type conversion (Database Engine) — Microsoft’s reference on implicit and explicit conversions, including the data type precedence rules that cause this issue.

### About Kevin

Kevin Griffin has been running production .NET applications and teaching developers for over two decades. A 16-time Microsoft MVP specializing in ASP.NET Core and Azure, Kevin brings real-world experience from his own SaaS products alongside his consulting work at Swift Kick. He's hosted the Hampton Roads .NET User Group since 2009 and founded RevolutionVA, the nonprofit behind Hampton Roads DevFest. Kevin's talks blend hard-won lessons from production systems with practical advice you can use Monday morning.

Categories
