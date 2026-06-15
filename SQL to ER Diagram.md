# SQL to ER Diagram

An open-source, client-side web tool that converts SQL CREATE TABLE statements into interactive entity-relationship diagrams. Paste SQL, drag tables, rename columns inline, add annotations, export to PNG/SVG — all in the browser, nothing uploaded. ~3,200 lines of vanilla JS with a single dependency (dagre for layout). Live at [sqltoerdiagram.com](https://sqltoerdiagram.com/).

---

## Architecture

A single-page app with 12 modules, no framework, Vite as build tool:

- **`src/parser.js`** (350 lines): SQL DDL parser. Tolerant of MySQL/PostgreSQL/SQLite/SQL Server dialects. Records source spans (character offsets) for every identifier to enable bidirectional canvas↔text editing.
- **`src/diagram.js`** (935 lines): Canvas rendering engine. Owns camera (pan/zoom), input handling, bitmap-cached table rendering, viewport culling, and annotation layer.
- **`src/renderer.js`** (210 lines): Table rasterization to offscreen bitmaps with theme support (dark/light).
- **`src/layout.js`** (152 lines): Auto-layout via `@dagrejs/dagre` with hub-aware edge weighting and orphan packing into compact grids.
- **`src/main.js`** (500 lines): App controller. Wires parsing → layout → rendering, manages localStorage persistence, handles the edit pipeline (canvas edit → surgical SQL rewrite → re-parse).
- **`src/edit.js`** (97 lines): Surgical text splicing using parser's source spans. Only changed bytes are touched — comments and formatting survive intact.
- **`src/share.js`** (51 lines): URL hash encoding with CompressionStream + base64url fallback. No server needed.
- **`src/svg-export.js`**, **`src/highlight.js`**, **`src/annotations.js`**, **`src/dialects.js`**, **`src/examples.js`**: Supporting modules for export, syntax highlighting, sticky notes, dialect type suggestions, and demo schema.

Data flow: SQL text → `parser.js` → model (tables + relations + spans) → `layout.js` (positions) → `diagram.js` (bitmaps + edges + annotations) → canvas. Edits reverse: canvas event → `edit.js` (surgical splice using spans) → SQL text update → re-parse (preserving positions).

## Key Techniques

### Source-Span Bidirectional Editing

The parser records exact character offsets for every identifier (table names, column names, column types, FK references). When you rename something on the canvas, `edit.js` splices only those bytes in the original SQL — preserving all comments, formatting, and unsupported clauses. Comments are replaced with equal-length whitespace during parsing (not stripped) so offsets remain valid 1:1 with the original text. This is the project's most underrated technical achievement — few diagram tools attempt true bidirectional editing.

### Canvas Bitmap Caching

Each table is rasterized once to an offscreen canvas, then blitted via `drawImage()`. The per-frame cost for hundreds of tables is just blits + edge drawing — no per-glyph text layout at render time. Tables outside the viewport are culled with a 50px margin.

### Hub-Aware Dagre Layout

Edges are weighted by endpoint table degree (capped at 12), so high-fan-out "hub" tables pull related tables into adjacent ranks. Orphan tables (no relationships) are packed into a compact grid beside the main graph rather than a single tall column. An O(n²) overlap removal pass (up to 60 iterations) handles edge cases dagre misses.

### Zero-Backend Sharing

Projects serialize to JSON → deflate-compress (CompressionStream API) → base64url → URL hash (`#s=…`). Browsers never send the hash to the server. Falls back to raw base64 when CompressionStream is unavailable. Clever and genuinely privacy-preserving.

### Dialect-Aware Type Autocomplete

Inline column type editing uses a `<datalist>` with type suggestions for the selected SQL dialect (PostgreSQL, MySQL, SQLite, SQL Server). Each dialect also provides a default type for new columns.

## Design Decisions

| Decision | For | Against |
|----------|-----|---------|
| Canvas rendering (vs SVG/DOM) | Smooth 60fps pan/zoom, bitmap caching scales to hundreds of tables | No accessibility, no text selection, custom hit-testing |
| Vanilla JS (vs React/Vue) | Tiny bundle, no framework churn | Manual state synchronization between editor and canvas |
| Tolerant parser (vs strict validation) | Works across MySQL/Postgres/SQLite/SQL Server | May silently accept malformed DDL |
| Zero-backend (vs server storage) | Privacy, no accounts, no infrastructure | No collaboration, URL length limits for very large schemas |
| Bitmap at fixed DPR (vs re-rasterize on zoom) | Smooth zoom interaction | Soft rendering at extreme zoom levels |

## Comparison Notes

- **vs [[sql-crack]]**: sql-crack visualizes SQL *queries* (execution flow, column lineage); SQL to ER Diagram visualizes SQL *schemas* (table relationships). Complementary tools.
- **vs [[HeidiSQL]]**: HeidiSQL is a full database client that connects to live databases; SQL to ER Diagram works from DDL text offline, with no connection required.
- **vs commercial ERD tools** (dbdiagram.io, drawsql.app): SQL to ER Diagram is the only one that's fully open-source, fully client-side with zero data leaving the browser, and free. The URL-hash sharing approach has no equivalent in commercial tools.
- **vs [[graphify]]**: Both transform structured text into visual graphs, but graphify targets codebases → knowledge graphs while SQL to ER Diagram targets DDL → ER diagrams.
- **vs [[Common Diagram Mistakes]]**: Pilger's catalogue of diagram anti-patterns applies to the output side — SQL to ER Diagram avoids several of them by default (all tables connected via FK edges, no isolated elements unless the schema genuinely has none).

## Why It Matters

SQL to ER Diagram demonstrates that a carefully designed single-purpose tool can compete with commercial alternatives through smart technical decisions: surgical text editing via source spans, bitmap caching for smooth interaction, and the URL hash as a zero-cost sharing backend. It's a masterclass in doing one thing well — the 3,200-line codebase packs more useful features than many tools ten times its size.

---

#tool #database #visualization #open-source

*Source: [[raw/sqltoerdiagram]]*
*Repository: https://github.com/royalbhati/sqltoerdiagram*
*Live: https://sqltoerdiagram.com/*
*Last updated: 2026-06-15*
