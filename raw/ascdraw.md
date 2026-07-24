---
url: https://github.com/exlee/ascdraw
title: ascdraw — Native, keyboard-first diagram editor on a Unicode canvas
author: Przemysław Alexander Kamiński (exlee)
date_fetched: 2026-07-25
date_published: 2026
---

# ascdraw

Native macOS/Unix desktop application for creating diagrams on an effectively infinite Unicode grid canvas. Written in Rust (~63K lines), it treats text as the native drawing medium: the user paints with Unicode box-drawing characters, arrows, symbols, and stamps using keyboard-driven modal editing.

## Architecture

Single monolithic native application with clean separation into:

### Core data model (`src/model.rs`)
- `Face` — styled cell appearance (fg, bg, underline, attributes like bold/italic/reverse)
- `StyledAtom` — a single display-width-1 grapheme with an assigned face
- `Atom` — validated single-grapheme cell content (validates display width == 1)
- `Coord` — {line: i16, column: i16} position on the infinite canvas
- `Direction` — {Up, Right, Down, Left}
- `ColorId` — 16 ANSI-style colors (8 base + 8 bright)
- `LayerId` — layer identifier with Greek-letter symbols (α through κ)
- Canvas limits: 20,000×20,000 max, 6 layers max

### Canvas — sparse layered grid (`src/canvas.rs`)
The fundamental data structure is `LayerMap`: each layer is a `BTreeMap<i16, BTreeMap<i16, CoordData>>` — a sparse row→column→cell map supporting negative coordinates and unbounded extents without allocating a dense grid.

Key canvas operations: set_at, delete_at, insert_cells, remove_cells, split_row, join_row_with_next, insert_column, insert_row, pull_column_left/right, remove_row, replace_bounds. All operations handle negative coordinates and i16 bounds.

Each `CoordData` holds an Rc<> shared `Face`, an Rc<> shared `Atom`, a `RefCell` raster cache, and optional `LineData` (line drawing connection metadata for routing).

`LayerStack` wraps a `Vec<LayerMap>` with an active index and enabled flag. Operations include: add_above, delete, merge_into, move_up/move_down, toggle_visibility. Layer 0 (the base layer) cannot be deleted.

### Document format (`src/document.rs`)
JSON sparse format with face deduplication: version 4. Sparse layers store cells as {line, column, face_id, atom, line_data} with faces in a separate indexed array. Supports legacy v3 format and even older TOML format via `src/legacy_loader.rs`. On save, coordinates are normalized to origin (0,0).

### Editor state machine (`src/editor.rs`, `src/editor/*.rs`)
The editor has a state machine with modes: StampMode, LineMode, TextMode, InsertMode, ReplaceMode, JumpMode, SelectionMode, MoveMode, LinePreviewMode, ShapePreviewMode.

Modal sub-editors:
- `line_tool.rs` — drawing with connected Unicode box-drawing characters using direction-bit connection tracking
- `line_preview.rs` — Space to start routing, move to route, Space to anchor, Space to finish
- `move_tool.rs` — MoveLift for moving selection blocks including clone-on-Shift
- `shape_tool.rs` — outline/filled rectangle preview
- `color_tool.rs` — color selection state
- `text_tool.rs` — text entry and replace modes
- `routing.rs` — line routing algorithm
- `jump_mode.rs` — spatial jump navigation (delegates to `src/jump.rs`)
- `utility.rs` — push/pull row/column operations

### Drawing engine (`src/drawing.rs`)
Maps direction bitmasks (UP=1, RIGHT=2, DOWN=4, LEFT=8) to Unicode box-drawing glyphs. Four line styles: Thin, Heavy, Double, Dashed. Two corner styles: Smooth (rounded corners ╭╮╰╯) and Sharp (┌┐└┘). Functions: `glyph_with_connection()` (add direction to existing glyph), `glyph_without_connection()` (remove direction), `is_line_glyph()` (check if character is a line-drawing glyph).

Glyph lookup is a pure function from {connections u8, style, corner_style} → char via exhaustive match.

### Rendering pipeline (`src/render.rs`, `src/render/*.rs`)
Uses Skia via `skia-safe` crate with `softbuffer` for framebuffer access.

Rendering flow per frame:
1. Acquire softbuffer pixel buffer
2. Wrap in Skia surface
3. Render toolbar (cached as raster image, cache key hashes toolbar state), grid atoms (sparse iteration with render-atom cache), selection overlay, cursor, jump overlay, minimap, tooltip
4. Present buffer

Three cache tiers:
- **Toolbar cache**: full toolbar raster, keyed by hash of toolbar state + theme + dimensions. Invalidated on any state change.
- **Rendered atom cache**: 512-entry LRU of individual pre-rendered cell images. Keys include the glyph, resolved colors, theme, and cell metrics. Prefill system rasterizes cells incrementally during idle with a 2ms budget per frame.
- **CoordData raster cache**: per-cell cached Skia images with generation tracking (style_generation ⊕ metrics_generation). Invalidated on theme/font/size changes.

Atlas batching: cells safe for atlas rendering (no custom background/underline/strikethrough, zero glyph overflow) are batched into `draw_atlas()` calls.

Fallback font system: detects non-ASCII characters, matches fallback fonts per character via Skia's `match_family_style_character()`, caches results.

Cursor rendering: Block, Beam, Underline shapes with adaptive contrast (WCAG relative luminance, minimum 3:1 contrast ratio, tries complement then black/white).

### Jump navigation (`src/jump.rs`)
Two-level spatial grid navigation:
- Level 1: divides visible viewport into 21×15-cell sector grid
- Arrow/hjkl move between sectors; moving past edge pans sectors
- After inactivity timeout, transitions to Level 2: 5×5 refined grid of 5×5-cell sectors
- After second timeout, lands at sector center (or selects from jump origin if Shift held)

### History/undo (`src/history.rs`)
256-entry undo stack using `VecDeque`. Supports grouped transactions:
- `ControlStroke` — all Ctrl+direction draws until Ctrl released are one undo entry
- `LineRoute` — entire line routing sequence is one undo entry
- `TextSession` — text insertion until leaving text mode is one undo entry

`HistoryCanvasDelta` records sparse before/after cells and layer topology changes. Merge operation combines deltas for grouped transactions.

### Automation protocol (`src/automation_protocol.rs`, `src/runtime/automation*.rs`)
Unix domain socket JSON-based protocol. Commands: Ping, Key (with modifiers/repeat/count), Text, Scroll, Zoom, State, Metrics, Screenshot, Shutdown. Request-response with id-based matching. Async dispatch via background thread.

### macOS integration (`src/macos.rs`, `src/render/metal.rs`)
- Native menus via objc2 (File, Edit, Window)
- Metal-backed rendering (optional, via skia-safe metal feature)
- App icon application
- Color space configuration for PNG export

### Input dispatch (`src/main.rs`)
The main event loop in `main.rs` (~3,600 lines) is a winit event loop handling keyboard, mouse, resize, IME, and automation events. Key classification uses `classify_key()` which considers editor state, cursor mode, modifiers, and key value to determine `KeyType` (history command, clipboard command, cancel, direction, jump, toolbar shortcut, edit command, or text input).

Modifier tracking (`OrderedModifierTracker`) records the *order* of modifier presses to distinguish Ctrl-then-Shift from Shift-then-Ctrl for correct 5/10-step actions.

### Export (`src/export.rs`)
`ExportPlatform` trait with native implementations for clipboard (text + PNG via arboard), file dialogs (via rfd), and PNG rendering (via Skia). Supports TXT, JSON (project format with version), PNG export. Can also load documents and import text into canvas.

### Toolbar (`src/toolbar.rs`, `src/toolbar/*.rs`)
The toolbar is a text-mode UI rendered into the top of the window. Modes: Stamp (stamps/symbols), Line (line drawing with style/ending selection), Shapes (rectangle drawing), Utilities (row/column push/pull, pan), Files/Togls (load/save/export, theme toggle, layer toggle, color toggle).

Toolbar state tracks: current main mode, submenu selections, pending shortcuts (digit prefixes), shift layer, color selection, layer operations, export menu state, document target. Durable menu selections are persisted in the JSON save format.

### Configuration (`ascdraw.toml`, `theme.toml`, `src/app.rs`)
Runtime-reloadable configuration via file watcher. Settings include font family, font size, theme colors (per-face: default, cursor, selection, toolbar elements, etc.), key bindings, jump inactivity timeout, macOS-specific settings.

## Key Techniques

1. **Sparse grid with negative coordinates**: Instead of a dense N×M array, each layer is a `BTreeMap<i16, BTreeMap<i16, CoordData>>`. This gives effectively infinite extent in all directions with zero allocation for empty cells. Line-drawing operations naturally handle negative strides.

2. **Direction-bit connection calculus**: Line glyphs are identified by their connection bitmask (a u8 where each bit means "connected in this direction"). Drawing extends connections by OR-ing direction bits and looking up the result in an exhaustive match table. This is purely functional — no graph traversal needed for individual glyph updates.

3. **Generation-based cache invalidation**: Rendering caches use `style_generation` (hash of default_face + theme) and `metrics_generation` (hash of font metrics) combined into a `raster_generation`. Cache entries carry their generation; mismatches trigger re-rendering. Theme changes or font size changes automatically invalidate all cached rasters.

4. **Atlas batching for same-glyph cells**: Cells sharing the same pre-rendered image that are "atlas safe" (no custom background, no decorations, no overflow) are batched into a single `canvas.draw_atlas()` call, which is significantly faster than individual `draw_image()` calls.

5. **Two-level spatial jump**: The jump system is a clever spatial navigation technique for large canvases. It creates an adaptive grid overlay, lets you navigate by sector, then automatically refines to finer sectors on inactivity. This lets you jump anywhere on a large canvas with ~3-5 keystrokes.

6. **Ordered modifier tracking**: Distinguishes Ctrl-then-Shift (5-step draw) from Shift-then-Ctrl (10-step draw) by tracking the press order of modifiers. This is handled by `OrderedModifierTracker` which records the sequence of modifier state changes.

## Design Decisions

- **Sparse over dense**: The fundamental choice to use `BTreeMap` layers over dense arrays. Trade-off: O(log n) cell access vs O(1), but unbounded canvas extent and minimal memory for empty regions. This is the right choice for a diagramming tool where content is sparse.

- **Skia over GPU-native**: Uses Skia (CPU rasterization for most operations, optional Metal backend on macOS) rather than DirectX/Metal/Vulkan directly. This gives consistent rendering across platforms at the cost of some GPU efficiency. The toolbar and atom caches compensate for the CPU rendering cost.

- **Text-mode toolbar over native UI widgets**: The toolbar is rendered as styled text spans using the same rendering pipeline as the canvas. This gives a cohesive monospace aesthetic but means the toolbar can't use native accessibility APIs or standard widget behaviors.

- **JSON sparse format with face dedup**: Rather than storing an explicit face per cell (which would duplicate hex color strings), the format assigns numeric face IDs and stores a separate face table. This is a pragmatic compression technique that also makes the format human-readable.

- **i16 coordinates with signed canvas**: Using i16 (−32,768 to 32,767) for coordinates is deliberately restrictive — it prevents runaway memory consumption while still providing a canvas thousands of times larger than any practical diagram. The 20,000×20,000 explicit limit provides an additional safety net.

- **Grouped undo transactions**: Rather than recording every keystroke as a separate undo entry, semantically related actions (control strokes, line routes, text sessions) are automatically grouped. This makes undo feel natural — undoing a line drawing removes the whole line, not one glyph at a time.

- **No GPU storage for canvas data**: All canvas data lives in CPU memory as BTreeMaps. Rendering iterates the visible subset and rasterizes on-demand. This keeps memory usage proportional to content, not canvas size.

- **Keyboard-first, mouse-tolerant**: The entire application is designed around keyboard shortcuts with vim-like mnemonics (hjkl movement, i for text, u/U for undo/redo). Mouse interaction is supported but treated as secondary — drag-select, click toolbar, scroll/zoom.

## Comparison Notes

Unlike **grok-mermaid** (which renders Mermaid diagrams to Unicode art in the browser), ascdraw is an interactive editor where the user draws directly on the Unicode grid. The output is not generated from a description language but constructed cell-by-cell.

Unlike GUI diagramming tools (draw.io, Excalidraw, Figma), ascdraw treats text as the primary medium. There is no vector graphics layer — everything is a Unicode cell. This means diagrams are inherently shareable as plain text (TXT export) and can be edited with any text editor as a fallback. It also means the tool can be used as a stdin/stdout filter (`ascdraw -`), integrating into vim/emacs/kakoune pipelines.

Unlike the [[Automatic Layout of Railroad Diagrams]] approach (where layout is a compilation problem), ascdraw gives the user full manual control over glyph placement. The line routing system provides assistance (auto-connecting glyphs) but the user positions everything.

The architecture shares some patterns with other Rust GUI applications: winit event loop, Skia rendering. But unlike eGUI or Druid which use retained-mode widget trees, ascdraw uses an immediate-mode approach — the toolbar is redrawn from state each frame (with caching for performance).

---

*Source: https://github.com/exlee/ascdraw*
*Last updated: 2026-07-25*
