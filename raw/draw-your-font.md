---
url: https://github.com/danilo-znamerovszkij/draw-your-font
title: draw-your-font
author: danilo-znamerovszkij
date_fetched: 2026-07-25
date_published: unknown
---

# draw-your-font

Turn a photo of your handwriting into a real font (TTF/WOFF/WOFF2) — free, open source, no uploads, no credits. MIT licensed, Node.js CLI + Claude Code skill + browser demo.

## Architecture

A deterministic image-to-font pipeline (~1,678 lines of JS across 12 source files) wrapped with an optional Claude Code skill layer for vision-based letter labeling and quality critique. The skill never draws letters; it only finds, labels, and judges them.

**Core pipeline** (each step a discrete module):
1. `capture.js` (62 lines) — Photo → binary ink mask via adaptive thresholding
2. `blob-core.js` (136 lines) — Connected components + multi-part glyph merging + reading-order clustering (pure typed arrays, no sharp/fs)
3. `segment.js` (83 lines) — Orchestrates capture + blob-core, writes numbered crops and contact sheets
4. `trace.js` (57 lines) — Crop PNG → SVG path data via potrace, with smooth/weight controls
5. `metrics.js` (79 lines) — Places traced glyphs into a shared 1000-UPM em square with per-character vertical bands
6. `winding.js` (107 lines) — Fixes potrace's even-odd contours to TrueType's nonzero-winding rule for correct counter rendering
7. `assemble.js` (62 lines) — Glyphs → SVG-font XML → svg2ttf → TTF/WOFF/WOFF2/CSS
8. `preview.js` (105 lines) — Render waterfall and glyph-sheet PNGs from vector paths (no font rasterizer)
9. `template.js` (65 lines) — Printable A4 PDF grid with light-grey guides erased by the adaptive threshold
10. `cli.js` (210 lines) — Commander-style CLI (`template`, `segment`, `build`, `make`, `preview`)
11. `charsets.js` (10 lines) — Two charsets: `minimal` (A-Z, a-z, 0-9, punctuation) and `spanish`
12. `index.js` (13 lines) — Library barrel export

**Dual runtime:**
- Node.js CLI: sharp for image processing, potrace for vectorization, svg2ttf/ttf2woff/wawoff2 for font binaries, pdfkit for template generation
- Browser demo (`site/demo.src.js`, 282 lines): Canvas-based capture reimplementation of `capture.js` (sharp is Node-only), `esm-potrace-wasm` for WASM potrace, shared `blob-core.js`/`metrics.js`/`assemble.js`

**Claude Code skill layer** (`skills/draw-your-font/SKILL.md`): A 141-line markdown file that teaches Claude to (1) run the CLI via a resolver script, (2) read contact sheets and write labels.json mapping blob-IDs to characters, (3) judge preview results like an art director, (4) iterate with refine flags (`--smooth`, `--weight`), and (5) provide legibility reports.

## Key Technical Details

### Adaptive binarization (capture.js)

Instead of a global threshold, the binarizer builds a local background estimate: heavy downscale (÷32), blur, rescale up. A pixel is ink if it's darker than both a global cap (165, to erase printed grey template guides) and its local background minus delta (40). This handles shadows, spiral binding, and uneven lighting without requiring the user to photograph perfectly. A morphological closing pass (dilate then erode) seals 1px threshold-flicker gaps along stroke edges.

### Multi-part glyph merging (blob-core.js)

The `mergeParts()` function handles three cases that would otherwise fragment a single letter into multiple blobs:
- **Stacked marks** (i-dot, colon, question mark): a tiny blob vertically above a stem, horizontally overlapping, with a vertical gap < 0.8× median glyph height
- **Side-by-side marks** (double quotes, equals): both tiny and horizontally close
- **Split strokes** (M, N, K drawn as disconnected lines): boxes intersect with significant vertical overlap, and the union stays character-width

The function recomputes median height each iteration (avoiding stale estimates from shadow specks that consolidate over rounds) and merges only the closest eligible pair per round to prevent a dot between two rows from joining the wrong stem.

### Reading-order clustering (blob-core.js)

`orderBlobs()` clusters blobs into rows by vertical overlap (robust to descenders like g/y that would confuse center-distance clustering), then left-to-right within each row.

### Per-character vertical bands (metrics.js)

The most important design decision. Instead of normalizing each glyph independently (which produces a ransom-note effect), every character is placed into a shared 1000-unit em square with per-character vertical bands defined as `[bottom, top]` in font coordinates:
- Capitals: [0, 700]
- Ascenders (b,d,f,h,k,l): [0, 720]
- x-height letters (a,c,e,m,n,o,r,s,u,v,w,x,z): [0, 480]
- Descenders (g,p,q,y): [-220, 480]
- Specific overrides for i, j, t, punctuation, diacritics, etc.

Each glyph's ink is scaled uniformly to fill its band. This shared coordinate system — not per-glyph normalization — is what makes the result feel like a font instead of a ransom note.

### Winding correction (winding.js)

TrueType fills with the NONZERO winding rule: outer contours must wind clockwise (in y-up font coordinates) and holes counter-clockwise. Potrace emits every contour in the same direction (it assumes even-odd fill), which makes counters — the bowls of b, g, o, a — fill solid in real renderers. `fixWinding()` parses SVG paths into subpaths, computes polygon area for winding direction, determines nesting depth via point-in-polygon tests, and reverses subpaths to achieve correct nonzero winding. Only M/L/C/Q/Z segments are handled, which is all potrace produces.

### Template grey-guide erasure

The printable template draws light-grey guidelines (#c8c8c8 cap line, x-height, baseline) inside each cell. The `--cap 165` default in the binarizer erases these — any pixel lighter than 165 in the normalised image is never ink. The user's dark pen (~#1c1c22) always falls below the cap, so only their writing survives.

### Weight adjustment (trace.js)

`adjustWeight()` is a pure morphological dilate/erode applied directly to the binary ink mask before tracing. No font interpolation; the stroke is literally grown or shrunk by N pixels in the crop image, then retraced. This produces genuinely bolder/thinner glyphs rather than stroking an outline.

## Innovations

- **Zero system dependencies**: No FontForge, no ImageMagick, no potrace binary. Everything is npm packages. Works on macOS/Linux/Windows wherever Node ≥ 18 runs.
- **Font authored as SVG-font XML**: `assemble.js` authors `<font>`, `<font-face>`, and `<glyph>` XML elements directly with metrics exactly as `metrics.js` computed them, then passes through svg2ttf. No icon-font "normalize" scaling anywhere in the chain.
- **Browser demo reuses the same modules**: The demo imports `blob-core.js`, `metrics.js`, and `assemble.js` directly. Only `capture.js` is reimplemented on Canvas (sharp is Node-only). This is a genuine shared core, not a port.
- **The skill file as infrastructure**: Rather than hardcoding vision prompts into the CLI, the skill file teaches Claude how to read the contact sheet, label blobs, judge output, and iterate — turning the AI into the UI. The deterministic pipeline remains AI-free.
- **Reading-order clustering by vertical overlap, not center distance**: Robust to ascenders and descenders that would pull center-points into adjacent rows.

## Design Trade-offs

- **No kerning, ligatures, or letter randomization** (v2 planned): The current output is a clean single-variant font. This simplifies the pipeline but means handwriting fonts lack the natural variation of real writing.
- **No AI in the trace step**: The CLI strictly uses deterministic potrace, never an AI model for vectorization. This guarantees consistent, predictable curves but means the font won't capture stylistic flourishes that potrace smooths away.
- **Photo quality sensitivity**: The adaptive threshold is clever but not magic — faint pencil, glossy paper, and severe shadows still cause problems. The troubleshooting guide is honest about this ("the honest fix is rewriting with a darker pen").
- **Client-side demo is Node-free but limited**: No WOFF2 in the browser (wawoff2 is Node-only), no template generation, and potrace via WASM is slower. But the demo proves the core pipeline can run entirely on the client.
- **No multi-character sequences**: Only single-codepoint characters are supported. Multi-codepoint sequences (ligatures, emoji ZWJ) are explicitly skipped.

## Dependencies

Node packages: pdfkit, potrace, sharp, svg2ttf, svgpath, ttf2woff, wawoff2
Dev: esbuild, esm-potrace-wasm, opentype.js
All pure npm. Zero native binary dependencies beyond what sharp bundles.

## Test Strategy

A single e2e test (`test/e2e.js`, 94 lines) that:
1. Generates a synthetic handwriting photo from stroked SVG paths (hand-drawn-looking A, b, g, i, o, x)
2. Runs the full CLI pipeline (`make photo --chars Abgiox --formats ttf,woff,woff2,css`)
3. Parses the output TTF back with opentype.js
4. Asserts: 6 glyphs found (i-dot merged), unitsPerEm=1000, correct per-character band placement (g descender < -100, b ascender > 650, o at x-height ≤ 500, A at cap > 650), space glyph at 300 advance, 'o' has outer+hole contours with opposite winding
5. Asserts woff2 magic bytes (wOF2), woff magic bytes (wOFF), @font-face CSS
6. Asserts all output artifacts exist (preview.png, glyphs.png, contact-1.png, blobs.json, manifest.json)

This is a live-fire test: it exercises the entire pipeline end-to-end and validates the most architecturally significant properties (shared metrics, winding correction) against a real parsed font.
