# FastBCP vs DuckDB: SQL Server to Parquet Benchmark

arpe.io's six-way benchmark exporting 150M TPC-H rows from SQL Server 2025 to zstd Parquet: FastBCP reading striped backup files in parallel wins at 10.1 s (18× DuckDB's 181 s), `RangeId` beats physical splitting 2.3× on a live columnstore table, and DuckDB is shown to be stream-starved rather than slow — all run, verified and written up by Claude Opus 5.5 using the vendor's own agent skill.

---

## What the benchmark shows

Six configurations, one laptop, one table. The headline result is architectural, not incremental: FastBCP's `backup_mssql` connector opens the striped `.bak` files directly with page-level random access, skipping SQL Server entirely. SQL Server sits at 0% CPU; the export no longer competes with production users, and it runs on any machine that can see the backup. The fastest live path (`RangeId`, 16 threads) is still 2.1× slower than the backup path at 21 s.

The second finding is about split strategy: at the same 16 threads, `RangeId` on `o_orderkey` beats `Physical` row-location splitting because columnstore segment elimination helps key ranges while row-location predicates make each query chew through far more data. Practical rule: prefer `RangeId`/`Ntile` on a good leading key over `Physical` on columnstore tables.

The third is the fairest single-stream comparison — same source, one stream, one file — where FastBCP's Arrow-native TDS driver still beats DuckDB's `mssql` extension by 2.7×. DuckDB's problem is diagnosed precisely: "starved, not slow" — about 0.6 core and SQL Server at 2%, flat-lined for three minutes, bottlenecked by its single read stream.

## Key quotes

> **"DuckDB is starved, not slow.** It uses about 0.6 core and SQL Server 2%, for 3 minutes. Its single read stream is the bottleneck."

This is the most useful sentence in the piece. It converts a benchmark into a diagnosis: DuckDB's 181 s is not an engine defect but a connector defect — one TDS stream into an otherwise parallel engine. The obvious fix (parallel streams) is exactly what FastBCP exists to do.

> **"On a live database, the split method matters more than the thread count."**

Counterintuitive and well-evidenced: `RangeId` and `Physical` use identical parallelism but land at 21 s vs 48 s, because where the CPU goes differs — 1,181 CPU-seconds in SQL Server for `Physical` against 280 for `RangeId`. Splitting is a query-planning problem, not a threading problem.

> **"The skill is what makes this practical... so the commands are right the first time instead of after a few failed runs."**

The meta-claim: FastBCP's option matrix (`backup_mssql` takes a directory in `--server`, `RangeId` needs `--distributekeycolumn`, `Physical` resolves differently on backups) is exactly the kind of tacit product knowledge a skill file should encode. The benchmark doubles as the skill's demo.

## The agent angle

The benchmark was designed, run and written up by Claude Opus 5.5 in Claude Code with the `fastbcp` skill: the agent wrote the PowerShell harness and sampler, ran 12 runs, cross-checked its metrics against FastBCP's self-reported CPU (agreement within 4%) and verified row counts and checksums. That last part is what separates this from an agentic flex — the agent's numbers had an independent oracle. It strengthens the pattern in [[Claude Code Skills System]] that skills encode domain rules on demand, and it shows agent-run benchmarks can be credible when verification is built into the loop. The diagnosis of where the time goes also complements [[What Is Columnar Storage (MotherDuck)]]: here the columnstore is both the reason `RangeId` wins and the reason the live path pays a decompression tax that the backup reader skips.

## Analysis

Honesty markers are unusually good for a vendor benchmark: caveats section up front about warm caches, shared NVMe, hybrid P/E cores; two passes per config; checksummed outputs; the cold single-stream pass reported separately (94 s vs 77 s). Still, the deck is stacked in a quiet way: the test table is a clustered columnstore, which the author himself notes penalises the live `RangeId` path — on a classic B-tree index each range would be a clean seek. And "18× faster than DuckDB" compares a parallel backup reader against a single-stream community extension; the honest headline is "the backup path skips half the pipeline," which the post does say, buried below the ×18.

The deeper point is that the default tool won because it is the default, not because it is wrong: DuckDB remains the right answer for a 10-second ad-hoc export, and FastBCP for a nightly 150M-row feed off a production box. The interesting question the post raises but doesn't answer: does DuckDB's `mssql` extension get parallel reads via ADBC?

---

*Sources: [[raw/fastbcp-vs-duckdb-sql-server-to-parquet-benchmark]], [[summary/fastbcp-vs-duckdb-sql-server-to-parquet-benchmark]]*
*Last updated: 2026-11-11*
