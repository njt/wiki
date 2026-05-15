---
title: "Dapper Performance Trap"
url: https://consultwithgriff.com/dapper-nvarchar-implicit-conversion-performance-trap
date_fetched: 2026-05-14
section: "C# and .NET"
---

# How C# Strings Silently Kill Your SQL Server Indexes in Dapper

## Core Problem

When passing C# `string` parameters through Dapper's anonymous objects to SQL Server `varchar` columns, a silent performance catastrophe occurs. Dapper defaults to mapping C# strings as `nvarchar(4000)` parameters. When these are compared against `varchar` columns, SQL Server must perform implicit conversion on every row, preventing index usage and forcing full table scans.

## The Mechanism

The issue stems from ADO.NET's default type mapping. When you pass a C# `string` through an anonymous object, Dapper maps it to `nvarchar(4000)`. This forces SQL Server to execute `CONVERT_IMPLICIT` operations, which disqualify indexed lookups. SQL Server's type precedence rules mean the column (VARCHAR) gets converted to match the parameter (NVARCHAR), not the other way around. This conversion happens per-row.

## Real-World Impact

The author discovered this destroying production performance -- a simple indexed query was consuming significant CPU resources. Benchmark results on a 1-million-row table: **176x slower** with default typing, and 268x slower on a 500,000-row table. Estimated query costs differed by over 1,000x according to SQL Server's own analysis.

## The Fix

Use `DynamicParameters` with explicit type specification:

```csharp
var parameters = new DynamicParameters();
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
```

Alternatively, use `DbString` with `IsAnsi = true` for inline definitions. Match parameter size to column definitions.

## Key Quotes

"A two-character type mismatch that was completely invisible in the C# code."

"If your column is `varchar`, SQL Server has to convert *every single value in the column* to `nvarchar`."

"The fix is almost embarrassingly simple."

## Conclusions

1. String parameters in anonymous objects targeting `varchar` columns create hidden performance disasters
2. Existing applications likely contain this issue invisibly degrading performance -- audit urgently
3. Document parameter type decisions with explanatory comments to prevent future refactors from reintroducing the problem
4. Simple rule: match ADO.NET parameter types to database column types; match parameter size to column size
