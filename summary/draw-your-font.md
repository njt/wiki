---
url: https://github.com/danilo-znamerovszkij/draw-your-font
title: "draw-your-font"
author: danilo-znamerovszkij
date_fetched: 2026-07-25
---

`draw-your-font` is a free, open-source (MIT) tool that turns a photo of
handwriting into a real TTF/WOFF/WOFF2 font — no uploads, no credits, no
FontForge dependency. It ships as a Node.js CLI plus a browser demo that shares
most of the same code, with an optional Claude Code skill layer for
vision-based letter labeling and quality critique.

The pipeline is a 12-module deterministic chain: adaptive binarization handles
uneven lighting and erases printed template guides; connected-component analysis
merges multi-part glyphs (i-dots, split strokes) and clusters blobs into
reading-order rows by vertical overlap; potrace traces each crop to SVG path
data with configurable smooth/weight controls; a winding corrector fixes
potrace's even-odd contours to TrueType's nonzero-winding rule so counters
render correctly; and metrics.js places every glyph into a shared 1000-UPM em
square with per-character vertical bands — capitals, ascenders, x-height
letters, and descenders each get their own zone. This shared coordinate system,
rather than per-glyph normalization, is what makes the result feel like a font
instead of a ransom note.

The Claude Code skill layer never draws letters; it only reads contact sheets,
writes label mappings, judges preview output like an art director, and
iterates with refine flags. The deterministic pipeline remains AI-free.

Notable trade-offs: no kerning, ligatures, or letter randomization (planned for
v2); no AI in the trace step (consistent but misses stylistic flourishes); and
photo quality matters — faint pencil or glossy paper still cause problems. The
project has a single live-fire e2e test that exercises the full pipeline and
validates the most architecturally significant properties (shared metrics,
winding correction) against a real parsed font.
