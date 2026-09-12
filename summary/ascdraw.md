---
url: https://github.com/exlee/ascdraw
title: "ascdraw — Native, keyboard-first diagram editor on a Unicode canvas"
author: Przemysław Alexander Kamiński (exlee)
date_fetched: 2026-07-25
date_published: 2026
topics:
  - developer-tools
---

ascdraw is a native macOS/Unix desktop application (~63K lines of Rust) for
creating diagrams directly on a Unicode grid. Instead of a vector graphics
layer, everything is a styled Unicode cell — box-drawing characters, arrows,
symbols, and stamps — making diagrams inherently shareable as plain text.

The canvas uses a sparse `BTreeMap` per layer (row → column → cell), supporting
negative coordinates and unbounded extent without allocating a dense grid. The
editor is modal and keyboard-first: Stamp, Line, Text, Insert, Replace, Jump,
Selection, Move, and preview modes, all with vim-like mnemonics (hjkl movement,
`i` for text, `u`/`U` for undo/redo). Mouse interaction is supported but
secondary.

Line drawing uses a direction-bit connection calculus: each glyph is identified
by a bitmask of connected directions (UP=1, RIGHT=2, DOWN=4, LEFT=8), and
drawing extends connections by OR-ing bits and looking up the result in an
exhaustive match table. Four line styles (Thin, Heavy, Double, Dashed) and two
corner styles (Smooth, Sharp) are supported.

Rendering uses Skia via `skia-safe` with `softbuffer` for framebuffer access.
Three cache tiers keep performance up: a toolbar raster cache, a 512-entry LRU
atom cache with incremental prefill, and per-cell raster caches invalidated by
generation tracking. Atlas batching groups same-glyph cells into a single draw
call. A two-level spatial jump system lets users navigate large canvases in ~3–5
keystrokes.

Undo uses a 256-entry stack with semantically grouped transactions — a full line
route, text session, or Ctrl+drag stroke is one undo entry, not a glyph-at-a-time
replay. Documents save as JSON with face deduplication (numeric face IDs + a
face table) and coordinate normalization to origin.

Architecturally, ascdraw is a single monolithic app with clean separation into
model, canvas, editor (state machine with modal sub-editors), drawing engine,
render pipeline, history, and export modules. A Unix domain socket JSON protocol
supports automation (key injection, screenshots, state queries), and the tool
can operate as a stdin/stdout filter for pipeline integration with vim, emacs,
or kakoune.
