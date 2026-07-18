---
url: https://github.com/malloydata/malloy
title: Malloy — A Semantic Modeling and Query Language
author: Malloy Contributors
date_fetched: 2026-07-18
date_published: 2021
---

# Malloy: A Semantic Modeling and Query Language Built on SQL

Malloy is an open-source language for describing data relationships and transformations. It functions as both a semantic modeling language and a query language that uses existing SQL engines (BigQuery, Snowflake, DuckDB, MotherDuck, PostgreSQL, MySQL, Trino, Presto, Databricks) to execute queries. Written in TypeScript as an npm workspaces/Lerna monorepo of ~93K source lines.

## Repository Architecture

The monorepo contains 17 packages under `packages/`:

| Package | Role | Lines |
|---------|------|-------|
| `malloy` | Core compiler, translator, runtime | 93,349 |
| `malloy-render` | Solid.js + Vega visualization layer | 22,018 |
| `malloy-interfaces` | Shared TypeScript types (Thrift-generated) | 4,364 |
| `malloy-db-duckdb` | DuckDB/WASM connection adapter | 4,393 |
| `malloy-filter` | Peggy-parsed filter expression compiler | 2,323 |
| `malloy-db-bigquery` | BigQuery adapter | 1,318 |
| `malloy-db-postgres` | PostgreSQL adapter | 1,229 |
| `malloy-tag` | Annotation/tag parser (MOTLY language) | - |
| `malloy-syntax-highlight` | Syntax highlighting | - |
| Others | MySQL, Snowflake, Trino, Databricks, Publisher adapters | - |

## Two-Phase Compilation Architecture

### Phase 1: Translator (`packages/malloy/src/lang/`)

ANTLR4-generated parser from `MalloyLexer.g4` and `MalloyParser.g4` grammar files produces a parse tree. The `MalloyToAST` visitor (`malloy-to-ast.ts`) transforms this into an AST, then into an Intermediate Representation (IR).

The IR is a **serializable data format** using plain objects, not class instances. It fully describes the semantic model independent of SQL. This means the IR can be cached, transmitted, and reused across compilations — a model file parsed once can generate SQL for any supported database without re-parsing.

Key files:
- `src/lang/grammar/MalloyLexer.g4`, `MalloyParser.g4` — ANTLR grammar
- `src/lang/malloy-to-ast.ts` — ~3000+ line ANTLR visitor translating parse tree to AST
- `src/lang/ast/` — AST node definitions
- `src/lang/malloy-to-stable-query.ts` — Converts to serializable query form

### Phase 2: Compiler (`packages/malloy/src/model/`)

Takes IR and generates SQL for specific database dialects. Produces SQL + metadata for result processing.

Key files:
- `src/model/query_query.ts` — Core query-to-SQL compilation, manages stage-based CTE generation
- `src/model/expression_compiler.ts` — Expression tree → SQL expressions, handles symmetric aggregates
- `src/model/stage_writer.ts` — CTE/UDF/PDT stage management
- `src/model/source_def_utils.ts` — Source identity and namespace resolution
- `src/model/malloy_types.ts` — ~700+ line type definition file with Expr union type and all IR node types

### Phase 3: API Layers (`packages/malloy/src/api/`)

Three API tiers:
- **Foundation** (`api/foundation/`): `Malloy`, `Model`, `PreparedQuery`, `Runtime` — the public classes
- **Stateless** (`api/stateless.ts`): Compile → SQL string without runtime state
- **Sessioned** (`api/sessioned.ts`): Connection-aware, can execute queries
- **Asynchronous** (`api/asynchronous.ts`): Streaming results via `EventStream`

The `Runtime` class wires everything together: config parsing, connection lookup, model loading, query compilation, and result materialization.

## Dialect System

The `Dialect` abstract class (`src/dialect/dialect.ts`) defines ~40+ boolean flags that each database adapter overrides:

```typescript
abstract class Dialect {
  abstract name: string;
  abstract defaultNumberType: string;
  abstract defaultDecimalType: string;
  abstract udfPrefix: string;
  abstract hasFinalStage: boolean;
  abstract divisionIsInteger: boolean;
  abstract supportsSumDistinctFunction: boolean;
  supportsAggDistinct: boolean;
  supportUnnestArrayAgg: boolean;
  supportsCTEinCoorelatedSubQueries: boolean;
  supportsQualify: boolean;
  supportsSafeCast: boolean;
  booleanType: BooleanTypeSupport; // 'supported' | 'simulated' | 'none'
  supportsNestedProjectionLimit: boolean;
  cantPartitionWindowFunctionsOnExpressions: boolean;
  hasLateralColumnAliasInSelect: boolean; // Databricks/Spark quirk
  // ... ~30 more
}
```

Each adapter (e.g., `src/dialect/pg_impl.ts`) extends `Dialect` and sets these flags + overrides SQL generation methods. This is a **trait/flag pattern** rather than an interface pattern — dialects don't implement every method, they inherit sensible defaults and override only what their database does differently.

Notable: the dialect system handles subtle SQL generation differences like:
- Whether `group_set` remapping needs a separate CTE (Databricks has lateral column aliases)
- Whether `FILTER (WHERE ...)` or `CASE WHEN ... THEN` syntax is used for aggregate turtles
- Integer type mapping: 32-bit ints → `'integer'`, larger → `'bigint'`, with per-dialect overrides (DuckDB has HUGEINT for 128-bit)
- Whether the dialect supports `STRUCT` nesting natively or needs JSON workarounds

## Key Data Structures

### Expr Union Type

The expression tree is a discriminated union of ~45 node types (`malloy_types.ts:68-111`). Key discriminator: `node` field (e.g., `'function_call'`, `'aggregate'`, `'field'`, `'filterCondition'`). Three structural shapes:

```
ExprLeaf     { node, typeDef?, sql? }
ExprE        extends ExprLeaf { e: Expr }
ExprWithKids extends ExprLeaf { kids: Record<string, Expr | Expr[]> }
```

### StructDef / SourceDef

`StructDef` represents any namespace-containing object (records, arrays, table schemas, query schemas). `SourceDef` is a `StructDef` usable as query input. The key distinction: a `SourceDef` has identity fields (`sourceID`, `referenceID`, `extends`) assigned in the translator and never propagated into compiled output — the compiler explicitly copies only needed fields, never using spread.

### SafeRecord<V>

A `Record<string, V>` wrapper with mandatory `safeRecordGet()` access. Purpose: prevent `Object.prototype` property pollution (`constructor`, `toString`, etc.) from being read as data fields. Uses `hasOwnProperty.call()` for safe property access.

## Symmetric Aggregates

The key semantic innovation. Consider `orders` joined to `line_items`. In SQL, `SUM(quantity)` over joined rows double-counts because each order appears N times for N line items. Malloy's solution:

```malloy
measure: total_quantity is line_items.quantity.sum()
```

The path prefix `line_items.` tells the compiler to aggregate at the line_item grain. The compiler's `isAsymmetricExpr()` guard function (`malloy_types.ts:184-189`) identifies `sum`, `avg`, `count`, `distinct` as needing symmetric handling.

In `expression_compiler.ts`, `sqlSumDistinct()` implements this by hashing a distinct key per row and using the dialect's `sqlSumDistinctHashedKey()` method:

```typescript
function sqlSumDistinct(dialect, sqlExp, sqlDistintKey) {
  const uniqueInt = dialect.sqlSumDistinctHashedKey(sqlDistintKey);
  const multiplier = 10 ** (precision - NUMERIC_DECIMAL_PRECISION);
  const safeValue = `CAST(COALESCE(${sqlExp}, 0) AS ${dialect.defaultDecimalType})`;
  // Scale, round, and sum distinct by hashed key
}
```

## Stage-Based SQL Generation

The `StageWriter` class (`stage_writer.ts`) orchestrates CTE generation:

```typescript
class StageWriter {
  withs: string[] = [];      // CTE bodies
  stageNames: string[] = []; // CTE names
  udfs: string[] = [];       // User-defined functions (for dialects that need them)
  pdts: string[] = [];       // Persistent derived table DDL
  stagePrefix = '__stage';
  stageNumber = 0;
}
```

Each query operation (group_by, aggregate, filter, nest) becomes a CTE stage. The `combineStages()` method stitches them into `WITH __stage0 AS (...) ... __stageN AS (...) SELECT ... FROM __stageN`.

For dialects that can't handle nested CTEs in correlated subqueries (`supportsCTEinCoorelatedSubQueries = false`), parameters are compiled into UDFs wrapped by `sqlCreateFunction()`.

## Render System

`packages/malloy-render` (22K lines) is the visualization layer.

### Architecture

Two renderer engines coexist:
- **New renderer** (`src/component/`): Solid.js reactive components, the default
- **Legacy renderer** (`src/html/`): HTML string builder, activated with `## renderer_legacy`

### Data Tree

Query results become two parallel trees:
- **Fields** (`fields/`): the schema — `RootField`, `RepeatedRecordField`, `ArrayField`, `RecordField`, atomic fields (`NumberField`, `StringField`, etc.)
- **Cells** (`cells/`): the data — parallel hierarchy, each cell knows its Field

### Dispatch

`RenderFieldMetadata` runs every plugin factory's `matches()` against each field and attaches the first hit. At render time, `applyRenderer` uses the matched plugin or falls back to the field's `renderAs()` value (computed from renderer tags).

### Plugin System

A factory-instance pattern:
- `RenderPluginFactory.matches(field, tag, fieldType)` → boolean
- `RenderPluginFactory.create(field)` → `RenderPluginInstance`
- Plugin instances implement `renderComponent()` (Solid.js JSX) or `renderToDOM()` (raw DOM)

### Tag System

Malloy annotations (`# tag`, `## model_tag`) are parsed by `packages/malloy-tag` using the MOTLY grammar. Two annotation routes:
- Route `''` — user-authored render hints (`# bar_chart`, `# currency`, `# label`)
- Route `malloy` — compiler-emitted metadata (`calculation`, `drill_*`, `query_timezone`, etc.)

The `malloy` route is a metadata side-channel: rather than widen the typed stable interface, core serializes structured metadata into annotation strings under `#(malloy)`, and the renderer reverses it.

### Vega Integration

Charts use Vega 5 (deliberately held — upgrading to Vega 6 requires a major renderer overhaul). `RenderResultMetadata` precompiles Vega runtimes. Cross-chart interaction (brushing, selection) uses a `ResultStore`.

## Configuration System

`malloy-config.json` supports:
- Connection definitions per database backend
- Overlay files for environment-specific config
- URL-based model discovery
- Inline model definitions (Malloy source embedded in config)

The config pipeline has three states (`api/foundation/config_compile.ts`):
1. Raw config (JSON parse)
2. Overlay-merged (layers applied)
3. Compiled (connections resolved, section compilers run)

## Code Generation Pipeline

```
Malloy Source (.malloy files)
  → ANTLR4 Parser (MalloyLexer.g4, MalloyParser.g4)
  → Parse Tree
  → MalloyToAST Visitor (malloy-to-ast.ts)
  → AST (ast/*.ts)
  → Translator (lang/malloy-to-ast.ts, lang/*)
  → IR (serializable, dialect-independent model representation)
  → QueryModel (query_model.ts, wraps IR for compilation)
  → QueryQuery (query_query.ts — walks query pipeline stages)
  → StageWriter (stage_writer.ts — CTE/UDF/PDP generation)
  → Dialect-specific SQL methods
  → SQL string + result metadata
  → Connection executes SQL
  → Result → Data Tree (renderer) → Visualization
```

## Design Decisions and Trade-offs

1. **IR as serializable data vs. live objects**: The IR is plain JSON, not class instances. This enables caching and transmission but means the compiler can't use methods on IR nodes — all logic lives in separate functions that pattern-match on `node` type.

2. **Trait-based dialect vs. full interface**: Each `Dialect` subclass overrides a few flags and methods rather than implementing every SQL generation method. This makes adding a new dialect fast (override ~10 things) but means behavior changes sometimes require tracing through flag combinations.

3. **CTE-heavy SQL generation**: Malloy generates deeply nested WITH clauses. This is readable and debuggable (each stage is named) but can hit query length limits on some databases and makes the generated SQL hard to hand-edit.

4. **Source-as-immutable-extension**: Sources are extended rather than mutated. Every `extend` creates a new source with the original plus additions. This enables composability but means source identity must be carefully managed across the compiler.

5. **Renderer as separate package**: The visualization layer is a separate npm package with its own build (Vite), its own test suite, and its own plugin API. This cleanly separates concerns but means the UMD bundle can't be loaded in Node without DOM stubs (Solid.js calls `delegateEvents()` at module-eval time).

6. **Explicit copy over spread**: The compiler explicitly copies fields from source definitions rather than using object spread. This prevents accidental propagation of identity fields (`sourceID`, `referenceID`) into compiled output — a deliberate guard against a class of bugs where source identity leaks into query results.

7. **Vega 5 lock**: The renderer pins Vega 5 while the ecosystem has moved to Vega 6. This is a deliberate hold because upgrading requires changes across the entire render stack — the vega-lite peer, chart runtime, and typings all need coordinated updates.

## Comparison to Related Tools

**vs. dbt**: dbt generates SQL views and tables from Jinja templates — it's a transformation layer. Malloy generates SQL at query time from a semantic model — it's a query layer. dbt models are compile-time artifacts; Malloy models are query-time artifacts. They're complementary: dbt could build the tables Malloy queries.

**vs. Looker/LookML**: Both are semantic modeling languages that generate SQL. Key differences: (1) Malloy is open-source and runs locally/in CI; Looker is a SaaS platform. (2) Malloy's query pipeline model (source → transform → source) is more composable than LookML's explores and views. (3) Malloy has native nested data support (arrays of records) as first-class syntax; LookML treats nesting as JSON manipulation.

**vs. Cube.js**: Both generate SQL from a semantic layer. Cube is API-server-first (REST, GraphQL, SQL API); Malloy is language-first (you write `.malloy` files, the compiler generates SQL). Cube has caching and pre-aggregation built in; Malloy delegates caching to the IR serialization format.

**vs. SQL itself**: Malloy is a higher-level language that compiles to SQL. It doesn't replace SQL — it organizes it. Raw SQL can be embedded in Malloy sources (`sql:""` blocks). The value is reusability: define a measure once, use it in any query.
