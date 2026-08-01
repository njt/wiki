---
url: https://visualeaf.com/blog/the-10-levels-of-building-a-data-grid/
title: "The 10 Levels of Building a Data Grid"
author: Ly Jacky Nhiayi
date_fetched: 2026-07-25
date_published: 2026-07-17
---

Ly Jacky Nhiayi walks through the performance optimizations needed to build a data grid that handles large SQL and NoSQL datasets with rich features: type-aware cells (BSON, JSONB), nested document expansion, in-place editing, and drag-and-drop into a visual query builder.

The journey starts with brute-force DOM rendering, which collapses at ~1,000 rows × 20 columns. The core fix is a **shadow table** — a precomputed data structure that resolves display strings, types, and flattened paths once, so rendering never computes per-cell formatting on every frame. On top of that, dual-axis virtualization (vertical via phantom-scroller slab, horizontal via prefix sums and binary search) keeps the rendered surface constant at roughly 40 rows × 12 columns regardless of data size.

Scroll-path discipline — passive listeners, coalesced `requestAnimationFrame` updates, a fast-draw exit when the viewport hasn't moved enough, and a velocity tracker that drops rendering during flicks — keeps the 16.7ms frame budget. Layout properties are then minimized: only `transform` and `opacity` animate on the compositor; zebra striping becomes a single `repeating-linear-gradient`; column-separator lines are clamped to the visible slab.

The most surprising win is **icon rendering**: per-cell icon elements with font-icon classes proved devastatingly expensive. Moving them to a `background-image` SVG data URI (shared across all cells of the same type) lets the GPU rasterize each icon once and blit it forever, with zero additional DOM nodes. Similarly, the heavyweight type-aware editor mounts only on the one cell being edited, via a portal, rather than on every cell.

DOM recycling via position-based identity tracking ensures scrolling never allocates — the same warm DOM nodes just get new values written into them. Finer touches include atomic reorder drops, lazy delegated tooltips, zebra-stripe parity keyed to absolute row index, and fixes for sub-pixel wobble.

The final level studies production grids: AG-Grid's time-sliced priority task queues and directional row building, and the tradeoffs of canvas-based grids (perfect 60fps but blurry text, limited selection, and no accessibility). The author chose the DOM path for text quality, real selections, accessibility, and development speed.
