# Hitomi (Data Viewer)

**Hitomi** is a Flutter desktop application for viewing and analyzing structured data — CSV, TSV, and arbitrary custom formats — with zero-wait startup regardless of file size. Its defining characteristic is a streaming ETL architecture: data is parsed, typed, and loaded incrementally as byte chunks arrive, so you can open a 10GB log file and see the first rows instantly. It also handles live data sources (pipes, sockets, subprocess stdout), and includes a custom expression-based filter language (PHLEX) with its own type system and compiler.

Why it matters: it's one of the few open-source desktop data viewers that treats streaming as a first-class architectural concern rather than bolting it on. The streaming parser correctly handles chunk boundaries cutting through multi-byte UTF-8 characters, escaped quotes, and CRLF pairs — edge cases that most "streaming" parsers ignore entirely. The filter engine has a selectivity-based caching strategy that functions as a mini query optimizer.

## Architecture

**Streaming ETL pipeline into a columnar in-memory store**, built in Dart/Flutter.

```
ByteStream >> Framer >> Scanner >> Columnarizer >> EvIntakeCtor >> EventManifold
```

Each stage is a Dart extension type with a `>>` operator, creating a readable pipeline syntax with zero wrapper-object allocation overhead.

- **ByteStream** (`lib/domain/byteStream/`): Abstracts file, socket, and subprocess I/O into `Stream<Uint8List>`. Three implementations: `QuiescentFileStream` (memory-maps), `TCPStream` (with reconnection), `SubprocStream` (subprocess stdout).

- **Framer** (`lib/domain/etl/Framer.dart`): Splits raw bytes into 32KB chunks to bound per-cycle memory.

- **Scanner** (`lib/domain/etl/Scanner.dart`): Dispatches to the appropriate parser based on schema type. CSV → `CSVparser`, TSV → `TSVparser`, custom formats → `Combinatorial` parser built from YAML grammar rules.

- **Columnarizer** (`lib/domain/etl/Columnarizer.dart`): Maps source fields to destination attributes, applying per-attribute transform chains (regex extraction → type parsing → default fallback).

- **EventIntake → EventManifold** (`lib/infra/amorphous/`): The EventManifold is the central data store — a `Map<String, SparseVector<Comparable?>>`, one column vector per attribute. On each chunk, it appends values and fires change callbacks.

- **View layer** (`lib/domain/backplane/`): `ViewsFactory` combines an `Index` (ordered/unordered integer sequence) with a column to produce typed views. `FilterSeries` maintains active filters and handles incremental updates when new data arrives.

- **UI layer** (`lib/presentation/`): Flutter Material Design. DataRails (virtualized data table, 608 lines), MiniMap (column overview, 737 lines), Unifinder (search), and tool panes for Analyze/Collect/Plot.

## Key Techniques

### Streaming parser with chunk-boundary recovery

The `Quadramaton` mixin (`lib/infra/quadramaton/Quadramaton.dart`) is a state machine that returns one of four states per call: `endOfEntry`, `endOfChunk`, `incomplete` (mid-field at chunk boundary), or `indefinite` (complete field but row terminator unseen). When `incomplete` fires, the partial row is discarded and the token stream rewinds to a checkpoint; parsing resumes correctly when the next chunk arrives. The test suite verifies: `'should handle incomplete UTF8 char at end of chunk'`, `'should handle CRLF split across chunks'`.

### PHLEX filter language with its own compiler

PHLEX (`lib/infra/phlex/`) is a full expression language: recursive-descent parser, 9 AST node types with visitor pattern, a type system (`ResultType` sealed hierarchy), function overload resolution with detailed error messages, and a three-pass compiler (parse → type-check → codegen) that produces row-index → bool closures. 14 built-in functions with 35 overloaded signatures.

### Selectivity-based filter caching

`FilterSeries` (`lib/infra/amorphous/FilterSeries.dart`) uses a dynamic strategy: if a filter's result is <50% of the original (high selectivity), cache it and run subsequent filters against the cached result — near-free. If >50% (low selectivity), apply against the already-filtered index. Uses O(m+n) two-pointer intersection to combine filter results, enabled by the ordering invariant that all filtered indices are subsequences of the original.

### Chunked lazy-allocation column storage

`SparseVector<T>` (`lib/infra/amorphous/SparseVector.dart`) uses 64-element chunks with a shared empty-chunk singleton. When all elements in a chunk equal the default value, the chunk reference is set to the shared empty chunk (no allocation). On write, the chunk is materialized. This means sparse columns with many null/default values consume almost no memory.

### Extension types for zero-cost pipeline abstraction

Dart 3 extension types let the code wrap configuration data (an int, a Schema, a list) with methods without allocating wrapper objects. `Framer(int size)`, `Scanner(Schema schema)`, `Columnarizer._(List)`, etc. are all compile-time-only wrappers. Combined with `>>` operator overloads, the pipeline syntax is both readable and allocation-free.

### Schema-driven attribute transform chains

Each `Attribute<T>` (`lib/domain/schema/Attribute.dart`) carries a `format` list of `FormatSegment` instances. A single segment → reshape (regex extraction). Multiple segments → assemble (build composite strings from captures + literal templates). The compiled `xformer` getter chains through type parsing with null/default fallback.

## Design Decisions

**Streaming over batch**: Zero-wait startup for any file size, at the cost of ~200 lines of parser state-machine complexity vs. ~50 lines for a batch CSV parser. The trade-off is unambiguous for a viewer — you never want to wait for a file to fully load before seeing anything.

**Columnar over row-oriented**: Column access is O(1) — right for sorting/filtering/plotting by column — but row access requires N independent vector lookups. A data viewer is column-access-heavy, so this is the correct choice.

**Custom query language over SQL**: PHLEX is simpler than embedding SQLite, operates directly on in-memory vectors without serialization, and gives meaningful type errors. But it's non-standard, lacks JOINs, and users who know SQL must learn a new syntax.

**In-memory only**: All data lives in SparseVectors in RAM. No spill-to-disk path. Reasonable for a desktop viewer where you're bounded by screen real estate before memory, but a 10GB file won't fit.

**Schema-first design**: Best results come from providing a schema file declaring attribute types and transforms. The schemaless fallback gives untyped string columns with no analysis capabilities — fine for quick peeks, useless for serious work.

## Comparison Notes

- **vs. VisiData**: Both are streaming-native data viewers, but VisiData is terminal-based (Python/curses) with vim keybindings and broader format support. Hitomi is GUI-based (Flutter) with a richer type system and a proper filter language with a compiler. Different audiences: terminal power users vs. GUI-first workflows.

- **vs. Tableau/PowerBI**: Hitomi is a viewer, not a BI platform. No dashboards, no data connectors beyond files/sockets/pipes. But open source, free, and instant startup.

- **vs. Excel/Google Sheets**: Hitomi is schema-driven and streaming-native, not cell-editing-oriented. Handles multi-GB files and live data sources that would crash a spreadsheet.

- **vs. [[Databases and Data|Databases & Data hub]] tools**: Unlike [[Streambed]] (CDC from Postgres to Iceberg) or [[Artie]] (managed CDC replication), Hitomi is an interactive viewer, not a data pipeline. Complementary: you could use Hitomi to inspect data flowing through a Streambed pipeline.

- The architecture echoes patterns from the [[Smart Models Dumb Pipes]] philosophy — the ETL pipeline is a "dumb pipe" (deterministic, code-owned) while the PHLEX filter language provides the "smart" interpretive layer. Unlike [[Apache Burr]]'s explicit state machine approach, Hitomi uses implicit state machines inside the streaming parsers and the Flutter widget tree.

Tags: #tool #desktop #database #data-analysis #flutter
[[databases-and-data]]

*Source: https://github.com/Verticalysis/Hitomi — ingested 2026-06-12 via deep code analysis*
