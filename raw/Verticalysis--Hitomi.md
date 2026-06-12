---
url: https://github.com/Verticalysis/Hitomi
title: Hitomi — Lightweight Streaming Data Viewer and Analyzer
author: Verticalysis
date_fetched: 2026-06-12
date_published: unknown
---

# Hitomi — Deep Architecture Analysis

## Summary

Hitomi is a Flutter-based desktop application for viewing and analyzing structured data (CSV, TSV, and arbitrary custom formats). Its defining characteristic is a **streaming ETL architecture** that provides zero-wait-time startup regardless of file size — data is parsed, transformed, and loaded incrementally as byte chunks arrive. It supports live data sources (pipes, sockets, subprocess stdout), custom parsers via a combinatorial parser framework (Quadramaton), a schema-driven typed attribute system, a custom expression-based filter language (PHLEX), and built-in analysis tools.

~68,600 lines of Dart across 95 source files. GPLv3 licensed. Runs on Windows, macOS, Linux, and web.

## File Tree (key source files)

```
lib/
  main.dart                          — CLI entry point, arg parsing, window init
  framework/                         — Shared utilities
    Collections.dart, EventDispatcher.dart, FileSystem.dart, EnhancedPatterns.dart
  infra/
    amorphous/                       — Column store and indexing
      Attribute.dart                 — Attribute type witness (not to be confused with domain/Attribute)
      EventIntake.dart               — Ingest wrapper for a stream, schema hinting
      EventManifold.dart             — Central data store: columns of SparseVectors, live updates
      FilterSeries.dart              — Multi-filter index with selectivity-based caching
      Index.dart                     — Ordered/Unordered/Desc/Skipped index abstractions
      IndexedView.dart               — Typed view combining an index with a column vector
      SortedList.dart                — Binary-search insertion into sorted lists
      SparseVector.dart              — Chunked lazy-allocation column storage
    phlex/                           — Custom filter expression language
      AST.dart                       — AST node types + visitor base class
      Parser.dart                    — Recursive descent parser
      Passes.dart                    — Type-checking and code-generation compilation passes
      Primitives.dart                — Built-in functions (len, time, relational ops, substr, regex)
      Types.dart                     — Type system (ResultType hierarchy, FunctionSignature, SymbolTable)
    quadramaton/                     — Streaming parser framework
      Quadramaton.dart               — Mixin: chunk-boundary-safe parsing state machine
      CSVparser.dart                 — Streaming CSV parser (RFC-4180 permissive)
      TSVparser.dart                 — Streaming TSV/WSV parser
      Combinatorial.dart             — Arbitrary format parser from CombinatorialPrimitive definitions
      TokenStream.dart               — Byte tokenizer with advance/consume primitives
      Encoding.dart                  — UTF-8/latin1 decoders as TokenStream tokenizers
    utils/
      TaggedMultiset.dart            — Set with lookup by custom key (used for schema attribute dedup)
      ListView.dart, ScalarTime.dart
  domain/
    analysis/                        — Analyzer plugin interface + built-in analyzers
      Analyzer.dart                  — Abstract ScalarAnalyzer/VectorAnalyzer interfaces
      builtin/                       — Integer, Numeric, Statistic, Regression analyzers
    backplane/                       — View layer between data store and UI
      Pipeline.dart                  — ETL pipeline assembler (connect schema + byte source)
      AttributesRegistry.dart        — Tracks attribute name→type mappings
      ViewsFactory.dart              — Constructs typed/sorted/stringfied views from indices
      StringfiedView.dart            — Typed column views with string conversions
    byteStream/                      — Byte source abstraction layer
      ByteStream.dart                — Abstract ByteStream, AddressFamily enum, InterruptNotifier
      QuiescentFileStream.dart       — File-backed lazy byte stream
      SocketStream.dart              — TCP socket byte stream with reconnection
      SubprocStream.dart             — Subprocess stdout byte stream
    etl/                             — ETL pipeline stages
      Framer.dart                    — Chunks raw bytes into fixed-size frames
      Scanner.dart                   — Dispatches to format-specific parser + adapter
      Columnarizer.dart              — Maps source fields→typed columns with xform chains
      Types.dart                     — ChunkedStream<T>, IntakeChunk typedefs
      adapter/                       — Adapts parser output to Columnarizer input
      schemaless/                    — Handles formats without a custom schema
    query/
      PhlexFilter.dart               — Compiles PHLEX expression→Filter closure
      FilterMode.dart                — Filter mode enum (row-based filtering)
      SearchController.dart          — Incremental text search across columns
    schema/
      Schema.dart                    — Sealed class: GenericSchema vs CustomSchema
      Attribute.dart                 — Typed attribute with transform chain and xformer getter
      AttrType.dart                  — Union-type enum with closures: from(), stringify(), allocVector()
      SchemasRegistry.dart, FlatDirSchList.dart
  presentation/
    application/                     — App shell, routing, theme
    workspace/                       — Main workspace: NavBar, MiniMap, DataRails, ToolPane, Unifinder
    widgets/                         — Shared widgets, dynamic chart builders
    launchpad/                       — File-open dialog
```

## Architecture Pattern

**Streaming ETL pipeline with columnar data store**

The core architecture is a linear pipeline where each stage is an extension type with a `>>` operator overload, creating a visually declarative flow:

```
ByteStream >> Framer >> Scanner >> Columnarizer >> EvIntakeCtor >> EventManifold
```

**Stage 1 — ByteStream** (`domain/byteStream/`): Abstracts file, socket, and subprocess I/O into a `Stream<Uint8List>`. Three implementations: `QuiescentFileStream` (memory-maps a file), `TCPStream` (with reconnection support), `SubprocStream` (subprocess stdout). Address resolution uses an enum-based factory pattern (`AddressFamily.file/net/cmd`).

**Stage 2 — Framer** (`domain/etl/Framer.dart`): Splits raw bytes into fixed-size chunks (32KB default). A simple pass-through — if a chunk is already under the limit, it passes unchanged; otherwise it slices into sub-chunks. This bounds memory usage per parse cycle.

**Stage 3 — Scanner** (`domain/etl/Scanner.dart`): Dispatches to the appropriate parser based on schema type and source format. For CSV → CSVparser, for TSV → TSVparser, for custom formats → Combinatorial parser. Each parser is wrapped in an adapter that normalizes output to `ChunkedStream<List<String?>>`. The Scanner is an extension type on Schema, making it look like a method call chain.

**Stage 4 — Columnarizer** (`domain/etl/Columnarizer.dart`): Maps source field positions to destination attributes, applying per-attribute transform chains. For schemaless mode, it's a bypass. For custom schemas, it builds `ColumnCtor` closures per attribute that extract the right source field index and apply the xformer pipeline (regex reshape → type parse → default fallback).

**Stage 5 — EventIntake/EvIntakeCtor** (`infra/amorphous/EventIntake.dart`): Wraps the columnarized stream as a sink that feeds into the EventManifold. Also carries schema type hints for attribute allocation.

**Stage 6 — EventManifold** (`infra/amorphous/EventManifold.dart`): The central data store. Maintains a `Map<String, SparseVector<Comparable?>>` — one sparse vector per attribute. On each incoming chunk of events, it appends values to each column vector, pads all vectors to the new length, and fires `onChange(newSize, oldSize)` and `onNewColumns(columns)` callbacks. Supports pause/resume/close for lifecycle control.

The `Pipeline` class (`domain/backplane/Pipeline.dart`) orchestrates all of this — it owns the EventManifold, creates the Scanner and Columnarizer for a given schema, and wires everything together when `connect(schema, byteStream)` is called.

### View Layer

Above the EventManifold sits a view layer:

- **Index** (`infra/amorphous/Index.dart`): Abstract integer sequence representing row ordering. Concrete types: `ListIndex` (mutable list), `OrderedIndex` (sorted by comparator), `DescIndex` (reversed), `SkippedIndex` (offset), and `PassthroughIndex` (identity — index[i] == i).

- **FilterSeries** (`infra/amorphous/FilterSeries.dart`): Maintains a chain of active filters applied to the index. When the data changes (`update(newSize, oldSize)`), new entries are run through all active filters and appended to the filtered index in order.

- **ViewsFactory** (`domain/backplane/ViewsFactory.dart`): Constructs typed or stringfied views by combining an Index with a column vector. Maintains a cache of OrderedIndex instances keyed by column name. The `ViewSwitcher` wraps this, handling the "sorted by X" state for the UI.

### UI Layer

Flutter/Material Design. The workspace has:
- **DataRails** — the main data table with virtualized scrolling
- **MiniMap** — a column overview/minimap (737 lines, the largest file)
- **Unifinder** — search/find widget
- **ToolPane** — contains Analyze, Collect, and Plotter sub-panes
- **NAV bar** — tab navigation between workspaces

## Key Techniques

### 1. Streaming Parser with Chunk-Boundary Recovery

The `Quadramaton` mixin (`infra/quadramaton/Quadramaton.dart`) is a state machine for streaming text parsing. Each call to `parseEntry()` returns one of four states:

- `endOfEntry` — complete row parsed, more data in this chunk
- `endOfChunk` — complete row parsed, chunk exhausted
- `incomplete` — last entry cut mid-field by chunk boundary (results discarded)
- `indefinite` — last entry complete but row terminator unseen (may resume)

The mixin maintains a `_commit` buffer and a `ckpt` position. When `incomplete` is returned, the token stream position is rewound to `ckpt` and the partial row is discarded. When the next chunk arrives and is appended to the same TokenStream, parsing resumes from the checkpoint. At end of stream, `finalize()` handles the terminal case: `incomplete` → error, `indefinite` → accept the last partial row.

The CSVparser (`infra/quadramaton/CSVparser.dart`) implements this interface with careful handling of quoted fields, escaped quotes, CRLF, and trailing commas. The non-strict mode is deliberately permissive (doesn't enforce RFC-4180's equal-field-count requirement).

### 2. Chunked SparseVector with Empty-Chunk Sharing

`SparseVector<T>` (`infra/amorphous/SparseVector.dart`) uses a two-level structure:
- A list of chunks, each being `List<T>` of size `2^chunkSizeExp` (default 64)
- A shared `_emptyChunk` singleton — when all elements in a chunk equal the default/empty value, the chunk reference is replaced with the shared empty chunk
- On write (`[]=`), if the target chunk is the shared empty chunk, a real chunk is allocated first

This is essentially a lazy-allocation trie for column storage — sparse columns (many null/default values) consume almost no memory per chunk, while dense columns allocate normally. The bit-shift addressing (`index >> _chunkSizeExp` for chunk, `index & (_chunkSize - 1)` for offset) is constant-time.

Also notable: `is`-based chunk identity checks (`_chunks[chunk] == _emptyChunk`) are O(1) rather than element-by-element comparison.

### 3. PHLEX — A Custom Filter Expression Language

PHLEX (`infra/phlex/`) is a complete expression language with:
- **Parser**: Recursive descent, ~200 lines (`Parser.dart`). Handles identifiers, literals (int, float, string), binary operators (`=`, `!=`, `<`, `>`, `<=`, `>=`, `in`), function calls, conjunctions (parens), disjunctions (square brackets), and negation (`!`).
- **AST**: 9 node types (`AST.dart`) with a visitor pattern — `ConjunctionExpr`, `DisjunctionExpr`, `InvocationExpr`, `InvertedExpr`, `BinaryExpr`, plus literals.
- **Type system** (`Types.dart`): `ResultType` sealed hierarchy — `BooleanResult`, `IntegerResult`, `FloatResult`, `StringResult`, `AbsoluteTimeResult`, `RelativeTimeResult`, `NumResult` (supertype of int/float), `VoidResult` (bottom type). `FunctionSignature` with `match(argTypes)` and `explainMismatch(argTypes)` for overload resolution.
- **Compilation passes** (`Passes.dart`): `TFApass` (Type-Function-Access) validates that all referenced identifiers have known types and all functions exist with matching signatures. `GenerationPass` compiles the typed AST into an `Artifact<T>` — a closure `T Function(int row)` that evaluates the expression for a given row index.
- **Primitives** (`Primitives.dart`): 14 built-in functions with multiple overloads (35 total signatures). Equality/inequality has 6 overloads each (including a catch-all `ResultType`/`ResultType` fallback that returns false/true respectively). Functions dispatched via a `FlattenedSignatureTable` — a flat list searched linearly per function name.

The compilation pipeline: `PHLEXparser.parse() → AST → TFApass.visit() → GenerationPass.visit() → Artifact<bool>` (a closure that takes a row index and returns true/false).

### 4. Selectivity-Based Filter Caching

`FilterSeries` (`infra/amorphous/FilterSeries.dart`) uses a dynamic strategy for combining multiple filters:

- When a new filter is added, it's first run against the **_original_ full index** (not the already-filtered result).
- If the result size is >50% of the original (`_overSelThresh`, threshold = 0.5), the result is **cached** as `_cachedResult` and this filter becomes `_cachedFilter`. This is the "high-selectivity" path — the filter eliminates most rows, so re-running from the original is cheap.
- If the result is ≤50% (low selectivity), it falls back to applying the filter against the already-filtered index. This avoids O(n) per filter when each filter passes most rows.
- When multiple non-cached filters exist, the result is the **intersection** of all filtered indices, computed via the O(m+n) two-pointer technique (`_intersectOrderedIndices`).

When new data arrives (`update()`), new entries flow through the cached filter first (if one exists), then through all other active filters. The filtered index is extended in-place.

### 5. Type-Safe Generic Attribute Pipeline

The `Attribute<T extends Comparable>` class (`domain/schema/Attribute.dart`) is parameterized by the concrete value type. But schemas are loaded dynamically from YAML, so the type is only known at runtime. The solution is a combination of:

- `AttrType` enum with stored closures (`from`, `stringify`, `allocVector`) — each enum value is a type witness
- `Attribute.relaxed()` constructor that does an unchecked cast (suppressing the type parameter constraint)
- `genericInvoke2/3` methods that call a generic function with `T` resolved at runtime
- An `AttributeFactory` extension on `AttrType` with a hardcoded constructor table indexed by `AttrType.index`

The `Columnarizer` uses `dstAttr.genericInvoke2(...)` to bridge from the runtime type to a `ColumnCtor<T>` with the correct type parameter, ensuring the transform chain produces typed values without casts inside the hot loop.

### 6. Attribute Transform Chains

Each `Attribute` has a `format` list of `FormatSegment` instances (StringSegment or RegExpSegment). The xformer getter distinguishes two cases:

- **Single segment** (`_vecXform == false`): Uses `_reshape` — calls `segment.transform(src)` which for RegExpSegment extracts capture groups, for StringSegment returns the literal string.
- **Multiple segments** (`_vecXform == true`): Uses `_assemble` — iterates all segments calling `segment.append(src, buffer)`, building a composite string. This supports patterns like: regex to extract a number, string literal to append units.

Both paths chain through `.then(parse)` where `parse` is the type-specific `from()` closure. Null values and parse failures fall back to `defVal`.

### 7. Combinatorial Parser Framework

`Quadramaton` has a general-purpose extension: `CombinatorialPrimitive` (`infra/quadramaton/Combinatorial.dart`) builds a parser from a YAML grammar definition. This allows custom file formats to be parsed without writing code — the schema file defines the format grammar declaratively.

The Scanner supports format strings: `"csv"`, `"wsv"`, `"annotated_csv"`, `"annotated_wsv"`, and `"Custom"` (the combinatorial parser). The `_customSetups` map dispatches to the appropriate parser+adapter combination.

### 8. Extension Types as Zero-Cost Wrappers

The codebase makes heavy use of Dart 3 extension types for zero-overhead abstraction:
- `Framer(int size)` — an int with a `process()` method
- `Scanner(Schema schema)` — a Schema with a `scan()` method
- `Columnarizer._(List<ColumnCtor>)` — a list with a `process()` method
- `EvIntakeCtor(Iterable<Attribute>)` — a schema list with a `construct()` method
- `PhlexFilter._(PHLEXexpr)` — an AST with an `asFilter()` method

Combined with the `>>` operator overloads, this creates a readable pipeline syntax without wrapper object allocation overhead.

## Innovation Points

**What would surprise a senior engineer:**

1. **The streaming parser is genuinely correct.** Many "streaming" parsers assume whole records fit in a buffer. Quadramaton handles the case where a chunk boundary cuts through a multi-byte UTF-8 character, through an escaped quote inside a quoted CSV field, or even through a CRLF pair. The test suite (`CSVparser_test.dart`) has specific tests for each of these scenarios. This is production-grade incremental parsing, not the usual "streaming in name only" approach.

2. **The filter series caching is a mini query optimizer.** The selectivity-based decision to cache or not, combined with the ordering invariant that enables O(m+n) intersection, means that adding a highly selective filter (e.g., `status = "error"` on a 1M-row table) makes subsequent filter additions nearly free — they run against the tiny cached result, not the full index.

3. **PHLEX is unusually complete for a desktop app.** Custom expression languages in desktop apps are usually regex-level or hand-rolled with severe limitations. PHLEX has a proper type system, function overload resolution with error reporting (showing which overloads were tried and why each failed), and a formal ABNF grammar. It's the quality level you'd expect in a database query engine, not a data viewer.

4. **Extension types eliminate entire allocation layers.** By making `Scanner`, `Framer`, `Columnarizer`, etc. extension types wrapping their configuration data, the `>>` chaining idiom (`src >> framer >> scanner >> columnarizer`) has zero intermediate object allocation — the extension type is erased at compile time. This is a genuinely clever use of a relatively new language feature.

## Design Trade-offs

- **Custom query language over SQL**: PHLEX is simpler and more domain-appropriate than embedding SQLite, but it's non-standard, has no JOINs, and any user who knows SQL has to learn a new syntax. The trade-off is that PHLEX can operate directly on in-memory SparseVector columns without serialization/deserialization overhead.

- **Streaming over batch**: The streaming architecture delivers zero-wait startup for any file size, but the cost is significant parser complexity. The CSV parser alone has to handle 4 states × multiple edge cases (quoted fields, CRLF, UTF-8, escaped quotes, empty fields, trailing commas). A batch parser would be ~50 lines.

- **Columnar over row-oriented**: The column store (one SparseVector per attribute) makes column access O(1) at the cost of making row access require index lookups across N vectors. This is right for a data viewer where you typically sort/filter/plot by column, but it means even simple operations like "show row 5" touch every column vector.

- **Dart/Flutter over native**: Cross-platform from a single codebase, but the Flutter rendering means it doesn't look native on any platform. The Material Design look is consistent but generic. For a data tool where screen real estate matters, platform-native table views might be more space-efficient.

- **Schema-first design**: The system works best with a schema file that declares attribute types and transforms. The schemaless path exists but gives you untyped string columns with no analysis capabilities. This is a deliberate choice to favor structured, well-understood data over quick-and-dirty CSV viewing.

- **No database backend**: All data lives in memory (in SparseVectors). This means instant filtering/sorting but limits dataset size to available RAM. For a desktop data viewer this is reasonable, but there's no spill-to-disk path for datasets larger than memory.

## Core Abstractions

1. **EventManifold** — The central data store. A column-oriented, append-only table implemented as `Map<String, SparseVector<Comparable?>>`. All reads (filtering, sorting, viewing) operate on this through the Index + View layer. All writes arrive via mounted EventIntakes.

2. **Attribute<T>** — A typed column descriptor. Knows its name, source field, type, transform chain, and default value. The `xformer` getter produces a `String? → T?` closure by composing reshape/assemble with type parsing. This is the bridge between raw text and typed data.

3. **Quadramaton** — A mixin providing the chunk-boundary-safe streaming parse loop. Implemented by CSVparser, TSVparser, and Combinatorial (the custom format parser). The 4-state return from `parseEntry()` is the core protocol.

4. **PHLEX Compiler** — A three-pass compiler (parse → type-check → codegen) that turns filter expressions into row-index → bool closures. The type system and overload resolution are sophisticated enough to give meaningful error messages.

## Type System (AttrType)

From `domain/schema/AttrType.dart` (not read in full, reconstructed from usage):

The AttrType enum carries closures for:
- `from(String)` — parse a string into the typed value
- `stringify(value)` — format a typed value for display
- `allocVector(int initialSize)` — create an appropriately-typed SparseVector

The index positions (used in `match()` and `AttributeFactory._constructors`):
0. ignore — columns marked as void (not displayed)
1. integer
2. float
3. hex — hexadecimal integer display
4. string
5. absoluteTime — timestamp
6. relativeTime — duration

## Comparison with Related Projects

- **vs. Excel/Google Sheets**: Hitomi is streaming-native and schema-driven, not cell-editing-oriented. It handles live data sources and multi-GB files without loading, but has no formula language for cell computation.
- **vs. Tableau/PowerBI**: Hitomi is a desktop app, not a BI platform. No dashboard building, no data source connectors beyond files/sockets/subprocesses. But it's open source, free, and starts instantly.
- **vs. VisiData**: Both are terminal/desktop data viewers with streaming, but VisiData is TUI-based (Python/curses) while Hitomi is GUI-based (Flutter). Hitomi has richer typing and a proper filter language; VisiData has vim-like keybindings and a broader set of format support.
- **vs. Datasette**: Simon Willison's SQLite-based data explorer for the web. Datasette has a proper query language (SQL) and plugin ecosystem; Hitomi has real-time streaming and custom format parsing. Different deployment models (web server vs desktop app).

## Code Stats

- ~68,600 total lines of Dart
- 95 Dart source files
- Largest file: MiniMap.dart (737 lines)
- Other large files: CellBuilder.dart (628), DataRails.dart (608), Plotter.dart (574)
- Core infrastructure files: 50-250 lines each
- Test coverage: Only Quadramaton tests (CSVparser, TSVparser, primitives) — no UI tests, no filter tests, no pipeline integration tests
