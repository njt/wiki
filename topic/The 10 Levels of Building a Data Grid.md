# The 10 Levels of Building a Data Grid

Ly Jacky Nhiayi's frontend performance engineering masterclass, disguised as a "how to build a data grid" tutorial. Walking through ten layers of optimization required to render a production-quality data table — from naive DOM loops through GPU-texture icon tricks to reading the source code of commercial competitors — the article is a practical taxonomy of browser rendering performance that applies far beyond data grids.

---

## Key Quotes

> "It died trying to render 1000 rows with around 20 columns."

The brute-force starting point. Each DOM node costs about a kilobyte; 300,000 nodes blows a 16.7ms frame budget instantly. The author's honesty about failure at a modest scale sets the tone — this is not a theoretical exercise.

> "Rows were only half the problem, and honestly... the other half was way harder to implement."

After vertical virtualization handles 10,000 rows, horizontal virtualization for 300 columns turns out to be the harder axis. Columns have variable widths, making prefix sums and binary search necessary where rows could get away with fixed-height arithmetic. A rare admission that the axis "nobody does" is the harder one.

> "A blank beat during a violent flick is imperceptible. A dropped frame is not."

The velocity tracker's design principle: above 10px/ms scroll speed, stop rendering entirely rather than drop a frame. This is performance engineering as perceptual psychology — optimizing for what the user notices, not what the profiler reports.

> "The browser rasterizes each icon exactly once and blits it from a cached GPU texture for every cell, every frame, forever."

The SVG data URI as background-image trick eliminates hundreds of DOM nodes. Every cell of the same type shares identical URI strings, so the GPU texture gets cached. The author calls it "the single best trick in the post" — warranted, because it's genuinely novel and underappreciated.

> "A framework component per cell is death by a thousand instantiations; a component per editing cell is one."

The hybrid rendering insight: cells are plain spans until double-clicked, at which point the heavy editor component portals in. This collapses the component count from O(n²) to O(1) in the editing dimension.

> "The grid works like an object pool: scrolling never allocates anything, the same warm DOM nodes just get new values written into them, no matter how far or how fast you scroll."

The recycling insight made concrete. Position-based tracking keys turn the DOM into a pre-warmed pool — the framework's expensive view-creation path fires exactly once per visible slot.

> "The single highest-value study habit in frontend performance."

The author's description of reading AG-Grid's source code. The meta-lesson: the best performance tricks aren't invented, they're discovered in open-source codebases that already solved your problem.

## Key Themes

#performance — The article is fundamentally about browser rendering performance, not data grids. Every level is a performance problem with a performance solution. The data grid is the vehicle, not the destination.

#virtualization — Levels 3 and 4 form the core: vertical virtualization (phantom scroller, window math, slab rendering) and horizontal virtualization (prefix sums, binary search, buffer margins). Together they make the rendered surface constant regardless of data size.

#dom-optimization — Levels 6 and 7 are about working with the GPU compositor rather than fighting it. The taxonomy of properties that animate on the compositor (transform, opacity) versus those that don't (everything else) is the single most actionable performance heuristic in the article.

#pattern — The shadow table (Level 2) is the deepest architectural insight: a precomputed display layer separate from raw data, carrying display strings, resolved types, and flattened paths. It's an ETL step inside the browser — transform-once, render-many.

#tool — VisuaLeaf is the author's product, but the article's value is entirely independent of it. The techniques are framework-agnostic and the source-code study habit (Level 10) recommends competitors.

#comparison — Level 10's DOM-vs-canvas comparison is concise and honest: canvas wins on raw throughput but loses on text quality, selection, accessibility, and development speed. The author picks DOM for the right reasons and doesn't pretend otherwise.

## Critical Analysis

**The shadow table is the real contribution.** Levels 3–5 (virtualization) are well-understood terrain — every virtual scrolling library implements them. But Level 2's shadow table is the architectural insight that makes everything else possible: precompute the display state once, then render from it. This decouples data format complexity (BSON types, nested paths, 16MB documents) from rendering performance. It's the kind of pattern that applies to any data-intensive UI, not just grids.

**The icon trick deserves to be famous.** SVG data URIs as background-image for per-cell type indicators is a genuinely underappreciated technique. It eliminates DOM nodes entirely for a common UI pattern, and the "every cell of the same type shares the identical string" property means browser caching handles deduplication automatically. This is the kind of optimization that feels like cheating once you see it.

**The velocity tracker is psychological engineering, not performance engineering.** Skipping rendering during fast scrolls is not about saving work — it's about perceptual masking. The author understands that users don't see dropped frames during flicks; they see jank during moderate-speed scrolling. This is taste, not technique, and it separates the article from generic performance advice.

**The article is a taxonomy, not a tutorial.** Each level names a problem class rather than providing implementation details. This is a strength — the patterns transfer to any framework — but readers expecting copy-paste code will be disappointed. The author assumes you can implement prefix sums and binary search once you know they're the solution.

**What's missing: accessibility and testing.** For a production data grid blog post, the absence of any discussion of screen reader support, keyboard navigation, or performance regression testing is notable. A grid that renders beautifully but can't be navigated by keyboard or announced by a screen reader isn't production-ready. This gap doesn't diminish the performance content, but it limits the article's claim to be about "building a data grid" rather than "making a data grid fast."

**The product pitch is unobtrusive but real.** The article closes with an invitation to try VisuaLeaf, and the custom-grid features (nested document expansion, type-aware search, query builder drag-and-drop) are the features that justify building rather than buying. It's honest marketing — show the engineering depth, then mention the product that contains it.

**The meta-lesson is studying source code.** Level 10's admission that the best techniques came from reading AG-Grid's source is the most valuable takeaway for engineers. The performance tricks that ship in commercial products are documented nowhere else; reading their source is genuinely "the single highest-value study habit in frontend performance." This applies far beyond data grids.

---
*Sources: [[raw/visualeaf-10-levels-data-grid]]*
*Last updated: 2026-07-25*
