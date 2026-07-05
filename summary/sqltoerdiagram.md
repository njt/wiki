---
url: https://github.com/royalbhati/sqltoerdiagram
title: SQL to ER Diagram
author: royalbhati
date_fetched: 2026-06-15
date_published: 2024
tags: [database, visualization, sql, erd, canvas, browser, open-source]
---

# SQL to ER Diagram — Full Analysis

Live hosted version: https://sqltoerdiagram.com/

## Overview

SQL to ER Diagram is a client-side web application that parses SQL CREATE TABLE / ALTER TABLE statements and renders interactive entity-relationship diagrams in an HTML canvas. It runs entirely in the browser — no backend, no accounts, no uploads. Users paste SQL, get an interactive diagram, drag tables around, edit names/types inline, add annotations (sticky notes, group boxes), and export to PNG/SVG. Projects can be saved/loaded as JSON files or shared via URL hash encoding.

The codebase is 3,176 lines of vanilla JavaScript across 12 source files, with one dependency (`@dagrejs/dagre` for graph layout) and Vite as the build tool.

## Architecture

The app follows a **single-page MVC-ish pattern** with clear module boundaries:

- **`parser.js`** (350 lines): SQL DDL parser. Extracts tables, columns, constraints, and foreign key relationships from CREATE TABLE and ALTER TABLE statements. Tolerant of MySQL, PostgreSQL, SQLite, and SQL Server dialects (backticks, double-quotes, bracket delimiters). Critically, it records **source spans** — absolute character offsets into the original SQL for every identifier (table names, column names, column types, FK references). These spans power bidirectional editing.

- **`diagram.js`** (935 lines): The core rendering and interaction controller. Owns the canvas, camera (pan/zoom with viewport culling), input handling (mouse events for table dragging, annotation manipulation, inline editing), and the render loop. Uses a dirty-flag + requestAnimationFrame pattern so it only repaints when something changes. Tables are rasterized once to offscreen bitmaps (via `renderer.js`) then blitted with `drawImage()`, so rendering hundreds of tables stays smooth.

- **`renderer.js`** (210 lines): Table rasterization. Measures text to compute table dimensions, then renders each table to an offscreen canvas with header, alternating row backgrounds, PK/FK badges, column names (left-aligned), and column types (right-aligned). Two themes (dark/light) with full color palettes.

- **`layout.js`** (152 lines): Auto-layout via dagre. Uses layered graph layout with hub-aware edge weighting (high-degree "hub" tables get stronger edge weights so spoke tables align in adjacent ranks). Orphan tables (no relationships) are packed into a compact grid beside the main graph rather than a tall single column. Includes an O(n²) overlap removal pass for dense schemas.

- **`main.js`** (500 lines): Application controller. Wires together parsing, layout, diagram rendering, and persistence. Handles the SQL textarea with a syntax-highlight overlay (transparent textarea on top of a `<pre>` with highlighted HTML). Manages localStorage persistence for SQL, layout, theme, and dialect preferences. Implements the edit pipeline: canvas edit → surgical SQL rewrite via spans → re-parse → update diagram (preserving table positions).

- **`edit.js`** (97 lines): Surgical text editing. Given a change from the canvas (rename table, rename column, change column type), uses the parser's source spans to splice only the affected bytes in the original SQL. Non-overlapping splices are applied right-to-left so earlier offsets stay valid. Can also insert new column definitions into CREATE TABLE bodies, preserving indentation style.

- **`share.js`** (51 lines): URL-encoded project sharing. Serializes the full project (SQL + layout + annotations + dialect) to JSON, optionally deflate-compresses via the CompressionStream API, base64url-encodes, and stores in `#s=` — the URL hash is never sent to the server. Falls back to raw base64 when CompressionStream is unavailable.

- **`svg-export.js`** (116 lines): Standalone SVG export. Rebuilds the diagram as SVG elements (rect, path for edges, text, circle) with theme colors. Handles word-wrapping for annotation text by character budget.

- **`highlight.js`** (55 lines): SQL syntax highlighting. Single regex pass per animation frame, producing span-wrapped HTML that glyph-aligns with the underlying textarea. Recognizes keywords, types, strings, comments, numbers, and quoted identifiers.

- **`annotations.js`** (52 lines): Sticky notes and group boxes. Four color palettes each, sanitization for loaded/shared data, unique ID generation.

- **`dialects.js`** (33 lines): Dialect definitions (PostgreSQL, MySQL, SQLite, SQL Server). Each provides a default column type and a list of type suggestions used in the inline type editor's `<datalist>`.

- **`examples.js`** (54 lines): A sample e-commerce schema (7 tables: users, addresses, products, orders, order_items, reviews) used as the initial demo.

- **`style.css`** (349 lines): All styling. Custom properties for themes, responsive layout, menu animations, inline editor positioning.

- **`index.html`** (222 lines): Single-page shell. Structured data (JSON-LD WebApplication + FAQPage), SEO-optimized with screen-reader-only crawlable content, Umami analytics (privacy-friendly, cookieless).

## Key Techniques

### 1. Source-Span-Preserving Parser (Bidirectional Editing)

The parser doesn't just extract a model — it records the exact character offsets of every identifier in the original SQL. When the user renames a column on the canvas, `edit.js` does a surgical text splice: only the bytes for that specific identifier change. Comments, formatting, and unsupported clauses are preserved verbatim.

```javascript
// From parser.js: column definition records name and type spans
const col = {
  name: colName,
  type: prettyType(type),
  typeRaw: type,
  pk: /\bprimary\s+key\b/.test(rest),
  nn: /\bnot\s+null\b/.test(rest),
  nameSpan: [ns, ne],        // absolute offsets for the bare identifier
  typeSpan,                   // absolute offsets for the type
};
```

Non-overlapping splices are applied right-to-left so earlier offsets stay valid:

```javascript
// From edit.js: applySplices
function applySplices(sql, splices) {
  const sorted = splices.slice().sort((a, b) => b.start - a.start);
  let out = sql;
  for (const s of sorted) {
    out = out.slice(0, s.start) + s.text + out.slice(s.end);
  }
  return out;
}
```

### 2. Comment Blanking for Offset Stability

Comments are replaced with equal-length whitespace (newlines preserved) rather than stripped. This keeps all character offsets in the "blanked" SQL identical to the original:

```javascript
// From parser.js: blankComments
// Replace '--' comments and '/* */' blocks with spaces, newlines preserved
if (c === '-' && c2 === '-') {
  while (i < n && sql[i] !== '\n') { out += ' '; i++; }
}
```

### 3. Canvas Bitmap Caching

Each table is rendered once to an offscreen canvas, then blitted via `drawImage()`. The per-frame cost for hundreds of tables is just drawImage calls + edge drawing — no per-glyph text layout at render time:

```javascript
// From diagram.js: _bitmap
_bitmap(t) {
  let bm = this.bitmaps.get(t.key);
  if (!bm) { bm = rasterizeTable(t, this.theme, this.dpr); this.bitmaps.set(t.key, bm); }
  return bm;
}
```

### 4. Hub-Aware Dagre Layout Tuning

Edges are weighted by the degree of their endpoint tables (capped at 12), so "hub" tables (high fan-in/fan-out) pull related tables into adjacent ranks rather than scattering them across the layout:

```javascript
// From layout.js
const hubness = Math.max(degree.get(from) || 0, degree.get(to) || 0);
const weight = 1 + Math.min(hubness, 12);
g.setEdge(from, to, { weight, minlen: 1 }, 'e' + e++);
```

### 5. Dialect-Aware Type Autocomplete

When editing a column type inline, a `<datalist>` element provides type suggestions based on the selected SQL dialect. Each dialect also has a default type for new columns (e.g., `text` for PostgreSQL, `varchar(255)` for MySQL).

### 6. Share Without Servers

Projects are JSON-serialized, optionally deflate-compressed via the CompressionStream API, base64url-encoded, and stored in the URL hash (`#s=…`). Browsers never send the hash to the server, so sharing requires no backend. Falls back to raw base64 when CompressionStream is unavailable.

### 7. Viewport Culling

The render loop computes the visible world-coordinate viewport and skips tables, edges, and annotations that lie entirely outside it (with a 50px margin). This keeps the render loop fast even with many tables zoomed in on a small area.

## Design Decisions

### Optimize for: Smooth interaction on large schemas → Trade off: Bitmap quality at extreme zoom

Tables are rasterized once at the device pixel ratio (capped at 2x). This means bitmaps look crisp at most zoom levels but may appear soft at extreme zoom. The trade-off is deliberate: re-rasterizing on every zoom change would make interaction janky.

### Optimize for: Zero-backend simplicity → Trade off: No collaboration, no persistence beyond localStorage

Everything runs in the browser. Share links encode the entire project in the URL hash. This means no accounts, no servers, no data retention concerns — but also no multi-user collaboration, no cloud sync, and URL length limits for very large schemas.

### Optimize for: Vanilla JS → Trade off: Manual state management

No React/Vue/Svelte. The app uses raw DOM manipulation with localStorage for persistence and Map-based caches for derived data. This keeps the bundle tiny and avoids framework churn, but means state synchronization between editor and canvas is manual (and carefully orchestrated).

### Optimize for: Tolerant parsing → Trade off: May accept malformed SQL

The parser is explicitly "tolerant of MySQL, Postgres and SQL-Server-ish dialects." It handles backticks, double-quotes, and bracket delimiters. This maximizes usability across SQL flavors but means it may silently accept some malformed DDL that a strict parser would reject.

### Optimize for: Canvas rendering → Trade off: No semantic DOM accessibility

Tables are rendered as pixels on a canvas, not as DOM elements. This enables smooth zooming/panning with bitmap caching but sacrifices screen-reader accessibility and text selection within the diagram area.

## Comparison Notes

- **vs sql-crack**: sql-crack is a VS Code extension that visualizes SQL *queries* as execution flow diagrams with column lineage tracing. SQL to ER Diagram visualizes SQL *schemas* (CREATE TABLE statements) as ER diagrams. They're complementary: sql-crack for understanding query execution, SQL to ER Diagram for understanding schema design.

- **vs HeidiSQL**: HeidiSQL is a full database client (query editor, table browser, import/export). SQL to ER Diagram is a single-purpose schema visualization tool. HeidiSQL can show ER diagrams but does so by connecting to a live database; SQL to ER Diagram works from DDL text without a connection.

- **vs commercial ERD tools (dbdiagram.io, drawsql.app, Lucidchart)**: SQL to ER Diagram is fully open-source, fully client-side (no data leaves the browser), and free. Commercial alternatives typically require accounts, store data server-side, or have paid tiers. SQL to ER Diagram's "share via URL hash" approach is genuinely novel among ERD tools.

- **vs graphify**: graphify builds knowledge graphs from codebases (code → multimodal graph). SQL to ER Diagram builds ER diagrams from SQL DDL (DDL → visual diagram). Different input types, different output types, but both transform structured text into visual graphs.

- The parser design (source spans for surgical editing) is reminiscent of how compilers track source locations for error reporting, applied here for bidirectional GUI↔text editing. This pattern is unusual in web-based diagram tools and is the project's most underrated technical achievement.
