# Dapper Performance Trap

When Dapper (the popular .NET micro-ORM) sends string parameters, it defaults to `nvarchar(4000)`. If your database column is `varchar`, SQL Server implicitly converts every row in the column to NVARCHAR before comparison -- defeating indexes and causing full table scans. The author's benchmarks show **176x slower** on a million-row table, with query cost estimates differing by over 1,000x. The fix is embarrassingly simple: use `DynamicParameters` with `DbType.AnsiString`.

---

## Key Quotes

> "A two-character type mismatch that was completely invisible in the C# code."

> "If your column is `varchar`, SQL Server has to convert *every single value in the column* to `nvarchar`."

> "The fix is almost embarrassingly simple."

## Key Themes

#dotnet #database #error-handling

The mechanism: ADO.NET maps C# strings to `nvarchar(4000)` by default. SQL Server's type precedence rules mean the column (VARCHAR) gets converted to match the parameter (NVARCHAR), not the other way around. This conversion happens per-row, which means the index is useless -- every row must be touched. The `CONVERT_IMPLICIT` operation shows up in execution plans but is easy to miss if you're not looking.

The fix:

```csharp
var parameters = new DynamicParameters();
parameters.Add("productCode", productCode, DbType.AnsiString, size: 100);
```

Or use `DbString` with `IsAnsi = true` for inline definitions. Match parameter size to column definitions.

The article emphasizes that this is invisible on small datasets and catastrophic on large ones -- 176x slower on a 1M-row table, 268x slower on a 500K-row table. The asymmetry between "it works fine in dev" and "it destroys production" is the defining characteristic of this class of bug.

This is particularly relevant in the age of AI-generated code. An LLM generating Dapper queries will produce the `nvarchar` default every time, because that's what the documentation shows. See [[dotnet Slopwatch]] for a tool that catches this kind of vibe-coding wart. The author's advice to document parameter type decisions with explanatory comments is also sound -- it prevents future refactors (human or agent) from reintroducing the problem.

## Critical Analysis

This is a well-executed diagnostic article: here's the problem, here's the benchmark proving it's real, here's the fix. The 176x benchmark number makes the case conclusively. The missing piece is a broader discussion of implicit conversion traps in other ORMs and languages -- similar problems exist in Entity Framework, Hibernate, and other data access layers. The real lesson is: always check your execution plans when queries are slower than expected, and never trust an ORM to generate optimal SQL. The article also underscores a general principle: type systems are contracts, and violating them silently (as ADO.NET does here) creates bugs that only surface under production load.

---
*Sources: [[raw/dapper-performance-trap]]*
*Last updated: 2026-05-14*
