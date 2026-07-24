---
url: https://visualeaf.com/blog/the-10-levels-of-building-a-data-grid/
title: The 10 Levels of Building a Data Grid
author: Ly Jacky Nhiayi
date_fetched: 2026-07-25
date_published: 2026-07-17
---

# The 10 Levels of Building a Data Grid

**Author:** Ly Jacky Nhiayi
**Published:** July 17, 2026
**Read Time:** 14 minutes
**Publication:** VisuaLeaf Blog

---

## Overview

This article details the author's design pattern for building a data grid that visualizes data for SQL and NoSQL databases. The grid required support for every BSON type and JSONB (with color-coded icons), expandable nested documents, reorderable/resizable/pinnable columns, in-place cell editing, drag-and-drop into a visual query builder, and search across nested paths with in-cell highlighting.

---

## Level 1. Brute Force: Render Everything

The naive approach: loop over rows, loop over fields, emit a cell. The author notes it "works at 100 rows" but at 10,000 rows with 30 columns (~300,000 DOM nodes), performance collapses. Each DOM node costs roughly a kilobyte in browser internals, and a 60fps frame budget of 16.7ms gets blown by any operation touching every node. In the author's own use case, "it died trying to render 1000 rows with around 20 columns." The solution: render only the visible slice.

## Level 2. The Shadow Table: The Foundation

Rather than reading raw documents during render (which forces per-cell formatting decisions on every frame), the author built a "shadow table" — a structure that reflects the data but **is** the state of the actual table. Raw documents remain untouched as the source of truth; the shadow table holds what the table shows. It precomputes per cell: the display string (truncated to prevent 16MB document leakage), the resolved type (for icons, editors, and search), and the flattened path (e.g., `address.geo.lat` as a single key instead of requiring tree walks). "Nothing about the DOM changed yet" — this was purely about data organization.

## Level 3. Vertical Virtualization: The Phantom Scroller

Three-part mechanism:

1. **The phantom** — an inner container sized to the full height of all rows (rowCount × rowHeight), giving the browser a real scrollbar for free.
2. **Window math** — on scroll: `firstRow = floor(scrollTop / rowHeight)`, `lastRow = floor((scrollTop + viewportHeight) / rowHeight)`, plus buffer rows on each side.
3. **The slab** — only the visible rows are rendered, positioned with a transform rather than `top`.

The user perceives a million-row table while "the DOM holds forty rows." However, with 300 fields, each of those 40 rows still renders 300 cells — bringing back thousands of nodes. "Rows were only half the problem, and honestly... the other half was way harder to implement."

## Level 4. Horizontal Virtualization: The Axis Nobody Does

Columns are harder than rows because widths vary. The author used two classic data structures:

- **Prefix sums** — a running total of column widths, so any column's x position is one array read.
- **Binary search** — to find which column sits at scroll offset x, binary-search the prefix sums in microseconds even with thousands of columns.

Prefix sums rebuild only when widths actually change (resize, hide, reorder), never during scroll. A fixed pixel margin (≈200px) buffers the window. "With both axes windowed, the rendered surface is constant: roughly forty rows by twelve columns" regardless of collection size.

## Level 5. Scroll-Path Discipline: 16 Milliseconds, Minus Everyone Else

Key disciplines used:

- **Passive listeners outside the framework** — the compositor can move pixels without waiting for JavaScript.
- **One coalesced update per frame** — a single `requestAnimationFrame` callback replaces handling every scroll event.
- **The fast-draw exit** (inspired by Handsontable) — if the newly computed visible range is still inside the rendered buffer, return immediately. Most ticks should cost "two integer comparisons and nothing else."
- **Hysteresis on buffer edges** — rebuild triggers when the visible range gets within ~40px of the buffer's edge, recentering the full 200px buffer. This prevents thrashing at boundaries.
- **The velocity tracker** — tracks pixels/ms between scroll events. Above a threshold (10px/ms, a flick), rendering stops entirely. "A blank beat during a violent flick is imperceptible. A dropped frame is not."

Despite the JavaScript going nearly silent during scrolling, frames still dropped. The bottleneck turned out to be styling: zebra striping, icons, layout, and paint — covered in the next levels.

## Level 6. Layout Properties Are a Tax

Only two properties animate on the compositor: **transform** and **opacity**. Everything else (top, left, width, height, background-color) wakes the main thread.

Frozen panels were syncing by updating `top` every frame — a forced layout 60 times/second. Switching to `translate3d` made "the visual identical and the main-thread cost zero."

**Zebra striping** became a single `repeating-linear-gradient` on the body, sized by a CSS variable for row height. "The browser rasterizes a single two-row tile and blits it from GPU texture memory forever."

**Column separator lines** that spanned the full phantom height (40 million pixels) were clamped to the rendered slab.

## Level 7. The Icons: The Most Surprising Optimization

Every cell needed a type indicator (ObjectId, string, int, JSONB). The initial approach — an icon element per cell with a font-icon class — proved "one of the most expensive decisions in the entire grid." Removing the icons made the table "SO MUCH smoother."

**The fix:** Icons became the cell's own `background-image`, encoded as an SVG data URI. Every cell of the same type shares the identical URI string, so "the browser rasterizes each icon exactly once and blits it from a cached GPU texture for every cell, every frame, forever." Zero additional DOM nodes. The author calls this the single best trick in the post.

**Hybrid rendering:** A cell is plain text (one span) until double-clicked, at which point the heavyweight type-aware editor mounts into that one cell via a portal. "A framework component per cell is death by a thousand instantiations; a component per editing cell is one."

## Level 8. Recycling vs. Identity: The Importance of Pooling

Frameworks control DOM identity via tracking functions (`trackBy` in Angular, `key` in React/Vue). Without proper tracking, old rows get destroyed and new ones created from scratch on every scroll — view creation being "one of the most expensive things a framework does."

The author tracks by **position** on both axes. "The grid works like an object pool: scrolling never allocates anything, the same warm DOM nodes just get new values written into them, no matter how far or how fast you scroll."

## Level 9. The Little Things

- **Atomic reorder drop** — during column drag, transforms shift columns; on drop, the reorder and transform reset land in the same render pass so "the last dragged frame and the first reordered frame are pixel-identical."
- **Lazy tooltips** — instead of a per-cell tooltip binding building strings on every render pass, one delegated hover listener resolves the tooltip for exactly the cell under the cursor.
- **Sub-pixel archaeology** — a one-pixel border on the row-number column caused a half-pixel baseline shift versus data rows, creating a visible wobble during scrolling. Fix: share a box model.
- **Zebra parity** — stripes keyed to the absolute row index, not DOM position, so colors never flicker as the window shifts.
- Unique features justifying a custom grid: nested documents expanding into real child columns, search highlighting matched substrings across nested paths, dragging typed values into a query builder.

## Level 10. Reading the Giants

The final level came from reading others' source code — "the single highest-value study habit in frontend performance." The author studied AG-Grid's approach: time-slicing DOM work through priority task queues with explicit per-frame budgets, hashing the column viewport so a no-op scroll costs one string comparison, building rows in the scroll direction, and deferring destruction until after creation so new cells paint before old ones disappear.

**Canvas grids** were also examined. They hold 60fps under any abuse by skipping the DOM entirely, but drawbacks include slightly blurry text (off pixel-grid rasterization), cell-at-a-time selection, and truncation without ellipsis. The author chose the DOM for "nice text, real selections, real accessibility, and development speed."

## Closing

The article concludes with an invitation to try **VisuaLeaf**, which includes a visual query builder for SQL and NoSQL, an ER diagram maker, table/collection management, and more — "all the SQL features are free."
