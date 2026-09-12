# ascdraw

A native Rust desktop application for drawing diagrams on an effectively infinite Unicode text grid. Keyboard-first, mouse-tolerant, renders at 120+ FPS via Skia. The premise: if you think in text, you should be able to diagram in text — with all the glyphs Unicode box-drawing provides, layered, colorized, and backed by full undo history.

---

## Architecture

ascdraw is a single monolithic Rust binary (~63K lines) with clean internal separation:

- **Canvas** (`src/canvas.rs`): The core data structure is a sparse grid per layer — `BTreeMap<i16, BTreeMap<i16, CoordData>>` — supporting negative coordinates, unbounded extent, and zero allocation for empty regions. Up to 6 layers with merge/delete/reorder operations.
- **Editor** (`src/editor.rs`, `src/editor/*.rs`): A modal state machine (Stamp, Line, Shapes, Text, Jump, Selection, Move) with specialized sub-editors for line drawing, shape preview, text entry, and selection moves.
- **Drawing engine** (`src/drawing.rs`): Direction-bit connection calculus — each line glyph is identified by a u8 bitmask (UP|RIGHT|DOWN|LEFT). Adding or removing connections is a pure functional lookup from {connections, style, corner_style} → char via exhaustive match. Four line styles (Thin, Heavy, Double, Dashed) with smooth or sharp corners.
- **Rendering** (`src/render.rs`): Skia-based pipeline with three cache tiers — toolbar raster cache, 512-entry rendered-atom LRU cache, and per-cell coordinated raster cache with generation-based invalidation. Atlas batching for same-glyph cells.
- **History** (`src/history.rs`): 256-entry undo stack with automatic grouping (ControlStroke, LineRoute, TextSession).
- **Automation protocol** (`src/automation_protocol.rs`): Unix domain socket JSON-RPC protocol for programmatic control.
- **Document format** (`src/document.rs`): Version-4 sparse JSON with face deduplication and origin normalization.

The main event loop (`src/main.rs`, 3,600 lines) is a winit event loop dispatching keyboard, mouse, resize, IME, and automation events through the editor state machine.

## Key Techniques

**Sparse grid with negative coordinates**: Each layer is `BTreeMap<i16, BTreeMap<i16, CoordData>>` — a sparse row→column→cell map. This gives an effectively infinite canvas in all directions without allocating a dense grid. Cell access is O(log n) per dimension, which is negligible for the visible subset rendered each frame.

**Direction-bit connection calculus** (`src/drawing.rs`): Line glyphs are identified by a bitmask of connected directions. The drawing system extends lines by OR-ing direction bits into the current glyph's connections and looking up the result in an exhaustive match table — no graph traversal needed for individual glyph updates. The entire line glyph space (4 styles × 2 corner styles × 16 connection patterns) is a set of pure functions.

**Generation-based cache invalidation** (`src/render.rs`): All rendering caches use generation keys: `style_generation` (hash of default_face + theme) and `metrics_generation` (hash of font metrics) combine into `raster_generation`. Cache entries carry their generation; mismatches trigger re-rendering. Theme or font changes automatically invalidate everything.

**Two-level spatial jump** (`src/jump.rs`): A spatial navigation system that overlays an adaptive grid on the viewport. Level 1 uses 21×15-cell sectors; after an inactivity timeout, it refines to 25 5×5-cell sectors centered on the selection. After a second timeout, it lands. 3-5 keystrokes to jump anywhere on a large canvas.

**Ordered modifier tracking**: Distinguishes Ctrl-then-Shift (5-step) from Shift-then-Ctrl (10-step) by recording modifier press order. This is handled by `OrderedModifierTracker` (`src/input.rs`) which tracks the sequence of modifier state changes.

**Atlas batching**: Cells sharing the same pre-rendered raster image that are "atlas safe" (default background, no decorations, no glyph overflow) are batched into a single `canvas.draw_atlas()` Skia call, dramatically reducing draw calls for large grids of repeated characters.

## Design Decisions

**Sparse over dense**: The fundamental architectural choice. `BTreeMap` layers trade O(1) cell access for unbounded extent and memory proportional to content. Right for diagramming where content is sparse and extent is unpredictable.

**Text-mode toolbar over native widgets**: The toolbar is rendered as styled text spans, giving a cohesive monospace aesthetic but losing native accessibility APIs and standard widget behaviors. An intentional constraint that keeps the entire UI in the same rendering pipeline.

**Skia CPU rasterization with Metal optional**: Uses Skia for cross-platform consistency. Toolbar and atom caches compensate for CPU rendering cost. Metal backend available on macOS but not required.

**JSON sparse format with face deduplication**: Numeric face IDs + separate face table instead of per-cell color strings. Pragmatic compression that preserves human-readability.

**Grouped undo transactions**: Semantically related actions (control strokes, line routes, text sessions) are automatically grouped into single undo entries. Undoing a line drawing removes the whole line, not one glyph at a time.

**i16 coordinates with safety limits**: i16 range (−32,768 to 32,767) prevents runaway memory while providing a canvas thousands of times larger than any practical diagram. The explicit 20,000×20,000 limit is an additional backstop.

**Keyboard-first, mouse-tolerant**: The app is designed around keyboard shortcuts with vim-like mnemonics (hjkl, i for text, u/U for undo/redo). Mouse interaction is supported but secondary. The app works as a stdin/stdout filter (`ascdraw -`), integrating into vim/emacs/kakoune pipelines.

## Comparison Notes

Unlike **[[grok-mermaid — Terminal Mermaid Renderer via WebAssembly]]**, which renders a Mermaid DSL to static Unicode art, ascdraw is an interactive editor where the user draws directly on the grid. The output is not generated from a declaration language but built cell-by-cell.

Unlike GUI diagramming tools (draw.io, Excalidraw, Figma), there is no vector graphics layer — everything is a Unicode cell. This makes diagrams inherently shareable as plain text and editable with any text editor. The `ascdraw -` filter mode makes it composable with text editor pipelines.

Unlike the **[[Automatic Layout of Railroad Diagrams]]** approach (layout as a compilation problem), ascdraw gives the user full manual control. The line routing system assists (auto-connecting glyphs) but the user positions everything.

The architecture shares patterns with other Rust GUI applications (winit + Skia pipeline) but uses an immediate-mode approach rather than retained-mode widget trees.

---

*Sources: [[raw/ascdraw]]*
*Last updated: 2026-07-25*
