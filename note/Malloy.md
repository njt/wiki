# Malloy

An open-source semantic modeling and query language that compiles to SQL. Created by ex-Google engineers (including the original Looker team), Malloy treats data models as first-class, reusable artifacts rather than one-off SQL queries. Every query you write extends the model for the next one.

---

## Architecture

Malloy is a TypeScript monorepo (~93K lines in the core package) organized as a **two-phase compiler**:

### Translation phase (`packages/malloy/src/lang/`)

Malloy source files (`.malloy`) are parsed by an ANTLR4-generated parser (`MalloyLexer.g4`, `MalloyParser.g4`). The `MalloyToAST` visitor class (~3000+ lines in `malloy-to-ast.ts`) walks the parse tree to produce an Abstract Syntax Tree, then transforms it into an **Intermediate Representation** — a JSON-serializable data format that fully describes the semantic model independent of any SQL dialect.

The IR is the system's most important architectural decision: it means a `.malloy` file is parsed once and the resulting IR can be cached, transmitted over the network, and compiled to SQL for any supported database. It also means the compiler and renderer never read Malloy source directly — they operate on IR.

### Compilation phase (`packages/malloy/src/model/`)

The compiler takes IR and generates database-specific SQL. Key components:

- **`query_query.ts`**: Walks query pipeline stages (group_by → aggregate → filter → nest) and orchestrates SQL generation
- **`stage_writer.ts`**: Manages CTE generation. Each pipeline operation becomes a named CTE (`__stage0`, `__stage1`, ...). For dialects without CTE-in-subquery support, compiles to UDFs instead
- **`expression_compiler.ts`**: Transforms Malloy's expression tree (45+ node types) into SQL expressions, handling symmetric aggregates, filter conditions, and type coercion
- **`dialect/dialect.ts`**: The abstract `Dialect` class with ~40 boolean flags that each database adapter overrides

### API layers (`packages/malloy/src/api/`)

Three tiered APIs: Foundation (raw compile), Stateless (compile to SQL string), and Sessioned (execute against a live connection). The `Runtime` class wires config → connection → model → compile → execute.

### Render pipeline (`packages/malloy-render/`)

A separate 22K-line package built on Solid.js and Vega 5. Query results become a dual tree of Fields (schema) and Cells (data). A plugin factory system (`matches()` → `create()`) dispatches visualization types. Annotations on model fields control rendering (`# bar_chart`, `# currency`, `# hidden`).

---

## Key Techniques

### Symmetric aggregates

The project's defining innovation: correct aggregation across joins without double-counting. In conventional SQL, joining `orders` to `line_items` and writing `SUM(quantity)` double-counts because each order row repeats N times. Malloy solves this with path-prefixed aggregation:

```malloy
measure: total_quantity is line_items.quantity.sum()
```

The path `line_items.` tells the compiler to aggregate at the line_item grain. The compiler's `isAsymmetricExpr()` identifies `sum`, `avg`, `count`, `distinct` as needing symmetric handling and injects distinct-key hashing (`expression_compiler.ts:sqlSumDistinct()`).

This is fundamentally different from SQL's approach: Malloy models joins as **properties of the source** (not per-query ad-hoc joins), and the aggregation grain is specified in the measure definition, not in the query structure.

### Trait-based dialect system

Rather than having each dialect implement a full interface, Malloy uses an abstract base class where dialects flip boolean flags and override a small number of methods. A new dialect typically overrides ~10 of the ~40+ flags plus ~5 SQL generation methods. Examples of flags:

- `supportsCTEinCoorelatedSubQueries` — if false, nested queries compile to UDFs
- `hasLateralColumnAliasInSelect` — Databricks quirk; forces extra CTE to avoid shadowing
- `cantPartitionWindowFunctionsOnExpressions` — forces dimension expressions into a lateral join bag
- `booleanType` — `'supported'`, `'simulated'` (converted to integers), or `'none'`

This is opinionated: it prioritizes fast dialect addition over clean abstraction. A new database can be supported in a few hundred lines of TypeScript.

### Pipeline query model (source as query output)

```malloy
source: base is table('raw_orders') extend { ... }
run: base -> { group_by: state; aggregate: order_count } -> { where: order_count > 10 }
```

Each `->` applies a query operation. Since query output is table-shaped, any query can be a source — this is how pipelining works recursively. The compiler models this as a `PipelineSegment[]` where each segment transforms the result of the previous one. This composability is what makes Malloy different from writing SQL: complex analytics emerge from chaining simple operations.

### Stage-based CTE generation

The `StageWriter` class (`stage_writer.ts`) manages a hierarchy of CTE writers. Each query stage compiles to a named CTE. The root writer collects all CTEs and produces a final `WITH __stage0 AS (...) ... SELECT ... FROM __stageN` query. Sub-queries get their own `StageWriter` instances that parent-link back to the root, so nested queries contribute CTEs to the root's WITH clause.

For dialects with `supportsCTEinCoorelatedSubQueries = false`, the compiler instead creates **user-defined functions** (UDFs) that wrap the inner query logic. This is handled transparently — the expression compiler doesn't know whether its output goes into a CTE or a UDF.

### Expression tree as discriminated union

Unlike most SQL compilers that use an object-oriented AST (each node type is a class), Malloy uses a tagged union: `Expr` is a type union of ~45 interfaces, each with a `node` discriminant string. Three structural shapes:

- `ExprLeaf` — terminal nodes with optional SQL literal
- `ExprE` — unary nodes wrapping a single child expression
- `ExprWithKids` — n-ary nodes with a `kids: Record<string, Expr>` map

Pattern matching is done via `switch (expr.node)` and type narrowing through the `node` discriminant. This is efficient for the compiler (no vtable dispatch) but verbose when adding new node types (must update the union definition, the SQL generator's switch, and the type-guard library).

### Tag metadata side-channel

Malloy's annotation system (`# tag`, `## model_tag`) serves double duty. User-facing: renderer hints (`# bar_chart`, `# currency`). Internal: a compiler → renderer metadata channel. Rather than widen the typed interface between compiler and renderer, the compiler serializes structured metadata (expression trees, timezone info, drill paths) into `#(malloy)` annotation strings. The renderer parses these back. This is pragmatic but fragile — the encoding is untyped at the interface boundary and versioned by convention.

---

## Design Decisions

### Optimized for: composability and reusability

Malloy's thesis is that a query language should accumulate knowledge. Define `on_time_rate` once in a source, use it in every query. Define `revenue` once, nest it in every breakdown. This compounding model is fundamentally different from SQL where each query is a fresh artifact.

### Sacrificed: query performance transparency

Because Malloy generates complex CTE chains, the resulting SQL can be 100–500 lines for what a human would write in 10 lines. Query optimizers handle CTEs well (especially DuckDB and BigQuery), but the generated SQL is effectively unreadable and un-debuggable by humans. The trade-off is correct: if you're debugging generated SQL, the abstraction has failed. But it's a cost — when a query is slow, understanding why requires tracing through the compiler, not just reading EXPLAIN output.

### Optimized for: dialect portability

The abstract `Dialect` class with flags means adding a new database is fast. But it's a leaky abstraction — some flags interact in surprising ways (e.g., `supportsCTEinCoorelatedSubQueries + hasFinalStage` changes the entire compilation strategy). The flags are documented in code but not systematically tested in combination.

### Sacrificed: IDE integration simplicity

Malloy uses ANTLR4 for parsing. ANTLR4 generates a large parser that's hard to tree-shake. The VS Code extension (in a separate repo) bundles the entire Malloy compiler. This makes the extension heavy (~10MB+) compared to a Tree-sitter-based approach (which would be ~1MB).

### Optimized for: renderer extensibility

The plugin system (`RenderPluginFactory.matches() → create()`) is clean and well-documented. Third-party renderers can be registered at runtime. However, the Vega 5 lock (deliberately held to avoid a major render stack upgrade) means the chart system is frozen on a major version behind the ecosystem.

### Sacrificed: bundle size and Node.js compatibility

The renderer is a Solid.js + Vega 5 UMD bundle. Solid.js calls `delegateEvents()` at module-eval time, which requires a DOM — the bundle crashes when loaded in Node.js. The `@malloydata/render-validator` works around this by stubbing `window`/`document`/`navigator` around its `require()`. This is a known cost of choosing Solid.js for a package that needs both browser and server rendering.

---

## Comparison Notes

Unlike **dbt** (which generates SQL views and tables at build time), Malloy generates SQL at **query time** — it's a query layer, not a transformation layer. dbt and Malloy are complementary: dbt builds the tables, Malloy queries them.

Unlike **Looker/LookML** (the SaaS platform), Malloy is open-source, runs locally, and doesn't require a server. Both share a semantic modeling philosophy (define once, query many times), but Malloy's pipeline model is more composable and its nested data handling is first-class syntax rather than JSON manipulation.

Unlike **Cube.js** (which is API-server-first with REST/GraphQL endpoints), Malloy is **language-first** — you write `.malloy` files, the compiler generates SQL, and you can execute that SQL however you want. Cube has caching and pre-aggregation built in; Malloy delegates caching to the IR serialization format and leaves execution to the caller.

The analogy that fits best: Malloy is to SQL what TypeScript is to JavaScript — a higher-level language with a type system, composable abstractions, and a compiler that generates well-formed lower-level code. [[Acadia]], Evan Czaplicki's Elm-style database language, occupies the neighbouring niche: Malloy is the analytics/query layer (define once, query many times), while Acadia is the application tier — table definitions, endpoints, and multi-step transactions compiled to SQL with compiler-verified migrations and end-to-end types. Both converge on "a real language that compiles to SQL"; they diverge on whether the source of truth is a semantic model or the program's own types.

A closely related compiler-in-the-loop architecture is [[Flint Chart]], Microsoft Research's visualization intermediate language. Both separate semantic modeling from backend code generation through an IR: Malloy compiles `.malloy` → IR → SQL; Flint compiles semantic types + data → `ChannelSemantics` → Vega-Lite/ECharts/Chart.js/Plotly/Excel. Malloy uses Vega for rendering; Flint could serve as a richer visualization layer for Malloy query output, offering multi-backend chart generation from the same semantic model.

---
*Sources: [[raw/malloy-semantic-layer]]*
*Last updated: 2026-08-06*
