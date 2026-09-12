---
url: https://peterbaile.github.io/beaver/
redirect_url: https://beaverbench.github.io/
title: "BEAVER: An Enterprise Benchmark for Text-to-SQL"
author: Peter Baile Chen, Devin Yang, Weiyue Li, Fabian Wenz, Yi Zhang, Nesime Tatbul, Michael Cafarella, Çağatay Demiralp, Michael Stonebraker
date_fetched: 2026-05-22
date_published: 2024-09-03
date_updated: 2026-05-13
source_type: paper
venue: arXiv:2409.02038v3 [cs.CL]
license: CC BY 4.0
topics:
  - databases-and-data
---

# BEAVER: An Enterprise Benchmark for Text-to-SQL

**Note:** The original URL (peterbaile.github.io/beaver) returns 404 as of May 2026. The project has moved to https://beaverbench.github.io/ and the GitHub repo at https://github.com/peterbaile/beaver has been archived (read-only, May 20, 2026). Content reconstructed from the arXiv paper (v3, May 2026) and the new project page.

## Abstract

BEAVER is the first text-to-SQL benchmark derived from private enterprise data warehouses. It comprises 9,128 question-SQL pairs sourced from real-world query logs, spanning 812 tables across 19 diverse domains. Existing benchmarks (Spider, BIRD) are built from public databases with clean schemas and simple queries; LLMs achieve ~82% on BIRD. BEAVER captures the reality of enterprise environments: hundreds of tables with cryptic column names, implicit join relationships, deep nesting, CTEs, window functions, and domain-specific conventions.

Key finding: State-of-the-art agentic frameworks using GPT-5.2 achieve only 10.8% execution accuracy on BEAVER. Even with all subtask annotations provided as oracle hints, accuracy reaches only 30.1% -- confirming fundamental bottlenecks in enterprise text-to-SQL that current approaches cannot solve.

## Motivation

Enterprise data warehouses differ fundamentally from public databases:
- Schemas contain hundreds/thousands of tables with obscure, abbreviated column names
- No explicit foreign key constraints -- join relationships are implicit
- Public LLMs have no prior exposure to domain-specific entities, codes, and conventions
- Queries are analytical: multiple joins, nested subqueries, CTEs, window functions

Existing benchmarks fail to capture this complexity. Spider 2.0 (the most challenging public benchmark) has 52.6 tables/DB on average and 2.9 tables per query. BEAVER has 101.5 tables/DB and 4.0 tables per query, with 5.7 joins and 3.7 CTEs per query on average.

## The Five Subtasks

BEAVER decomposes text-to-SQL into five annotated subtasks, enabling fine-grained diagnosis beyond binary execution accuracy:

1. **Multi-table retrieval** -- Identify the correct set of connected tables from a large schema
2. **Join key detection** -- Determine how tables connect via overlapping/semantically related columns (no FK constraints in enterprise DBs)
3. **Column mapping** -- Map NL phrases to table columns with opaque names (e.g., "rooms" → FCLT_ROOMS.FCLT_ROOM_KEY)
4. **Domain knowledge extraction** -- Map NL entities to instance values (e.g., "Stata building" → BUILDING_KEY = 32)
5. **Query decomposition** -- Determine whether to decompose into CTEs/sub-queries vs. generating SQL directly

All five subtasks are annotated by domain experts (6 grad students + 6 DBAs). Inter-annotator agreement: ~87%.

## Structural Template Recomposition

Only 254 real queries were obtainable from query logs due to privacy constraints. BEAVER's key innovation is **Structural Template Recomposition** (STR), a pipeline that:

1. Extracts atomic structural templates from real queries (base, nesting, CTE, domain knowledge templates)
2. De-duplicates and retains frequent templates (594 total extracted)
3. Composes templates iteratively using GPT-5.2 for initial drafts with expert verification at each step
4. Creates 8,874 synthetic queries that isolate individual challenges

Three query categories emerge:
- Complex queries without domain knowledge
- Domain-specific queries with low complexity
- Complex queries with domain knowledge (the hardest category)

Real and synthetic queries show comparable difficulty, validating synthesis fidelity.

## Dataset Statistics

| | BEAVER | Spider 2.0 | BIRD |
|---|---|---|---|
| Tables/DB | 101.5 | 52.6 | 6.8 |
| Columns/DB | 869.4 | 803.6 | 72.5 |
| Test queries | 9,128 | 547 | 1,534 |
| Tokens/query | 316.7 | 144.5 | 37.3 |
| Tables/query | 4.0 | 2.9 | 2.0 |
| Joins/query | 5.7 | 2.4 | 0.9 |
| Functions/query | 8.0 | 6.5 | 1.3 |
| Nesting levels | 5.6 | 4.9 | 1.1 |
| CTEs/query | 3.7 | 2.5 | 0.0 |
| Annotated subtasks | 5/5 | 1/5 | 1/5 |

## Evaluation Results

### End-to-End Performance (Execution Accuracy %)

| Method | Best Model | BEAVER | Spider 2.0 |
|---|---|---|---|
| ReFoRCE | Claude-4.5-sonnet | 11.4 | 62.9 |
| Few-shot | GPT-o4-mini | 8.7 | — |
| DIN-SQL | GPT-5.2 | 6.1 | — |
| DAIL-SQL | GPT-5.2 | 4.7 | — |

ReFoRCE with Claude-4.5-sonnet scores 11.4% on BEAVER vs. 62.9% on Spider 2.0 -- a 5.5x gap, confirming that enterprise text-to-SQL is fundamentally harder.

### By Query Category (averaged across all models)

| Method | Complex (no DK) | Domain-specific | Complex + DK | Real Queries |
|---|---|---|---|---|
| ReFoRCE | 7.2 | 32.0 | 5.4 | 8.2 |
| Few-shot | 5.0 | 30.8 | 3.7 | 5.8 |

Complex + domain knowledge is the hardest category (5.4% for the best method).

### With Subtask Annotations (ReFoRCE)

| Setting | Exec Acc | Table F1 | Join F1 | Col Map F1 | DK F1 | Decomp |
|---|---|---|---|---|---|---|
| End-to-end | 9.5 | 70.1 | 35.5 | 61.6 | 20.4 | 50.8 |
| + Schema linking hints | 18.9 | 91.3 | 71.9 | 82.1 | 23.9 | 58.3 |
| + All subtask hints | 30.1 | 91.9 | 75.7 | 83.4 | 27.1 | 64.8 |

Even with oracle hints for all five subtasks, execution accuracy maxes out at 30.1%. Schema-linking subtasks (table retrieval, join keys, column mapping) are largely solvable with hints. Domain knowledge and decomposition remain stubbornly hard, even with hints.

## Error Analysis

Analysis of 275 failed queries with all subtask hints reveals three error categories:

### Analytical & Structural Errors (46.8%)
- **Grouping/ordering (22.8%):** Missing GROUP BY/ORDER BY, wrong aggregation granularity
- **Advanced functions (14.2%):** Wrong PARTITION BY, incorrect window frame bounds, dropping EXP/LN in geometric mean computation
- **Query decomposition (7.7%):** Using flat GROUP BY instead of ROLLUP for subtotals
- **Row selection (2.1%):** Missing DISTINCT, incorrect LIMIT

### Schema Linking Errors (35.2%)
- Column selection (12.5%), table selection (11.8%), join conditions (10.9%)

### Predicate Errors (18.0%)
- Missing predicates entirely (10.6%), incorrect literal values (7.4%)

## Databases

Three private enterprise data warehouses:
- **DW:** 97 tables, 1,530 columns -- Oracle DB, academic institution (courses, students, buildings, rooms)
- **NW:** 366 tables, 2,708 columns -- MySQL, research lab infrastructure (compute, virtualization, IAM)
- **SP:** 349 tables, 2,717 columns -- MySQL, residential facility management (check-in, packages, events)

All publicly released with instance-level anonymization.

## Models Tested

Proprietary: GPT-o4-mini, GPT-5.2, GPT-5-mini, Claude-4.5-sonnet, Gemini-2.0-flash
Open-source: Qwen3-Next-80B-A3B-Instruct, MiniMax M2.1

Methods: ReFoRCE (agentic), DIN-SQL (decomposition + self-correction), DAIL-SQL (prompt engineering), few-shot
