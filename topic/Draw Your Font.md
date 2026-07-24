# Draw Your Font

A Node.js CLI and Claude Code skill that turns a photo of your handwriting into a real installable font (TTF/WOFF/WOFF2) — free, open source, zero uploads, no system dependencies beyond Node ≥ 18. The pipeline is deterministic image processing (adaptive threshold → blob detection → potrace vectorization → per-character metrics → font assembly); the AI layer only finds, labels, and judges letters — it never draws them.

---

## Architecture

A 12-module, ~1,678-line Node.js pipeline with a dual runtime strategy (CLI + browser demo sharing 4 core modules) and an optional Claude Code skill layer.

```
photo → [capture.js] → ink mask → [blob-core.js] → ordered blobs
     → [trace.js] → SVG paths → [metrics.js] → placed glyphs
     → [winding.js] → TrueType-correct contours
     → [assemble.js] → TTF/WOFF/WOFF2/CSS
```

**The pipeline is purely deterministic.** The skill file (`skills/draw-your-font/SKILL.md`, 141 lines) teaches Claude to read the numbered contact sheet from the segment step, write a `labels.json` mapping blob IDs to characters, then judge the preview output like an art director. Claude sees and judges; the CLI does all geometry. This is the [[Smart Models Dumb Pipes]] pattern applied to font creation: the end-to-end principle where smart models own decisions (labeling, quality judgment) and dumb pipes own execution (trace, metrics, assembly).

The browser demo (`site/demo.src.js`) shares `blob-core.js`, `metrics.js`, and `assemble.js` directly — only `capture.js` is reimplemented on Canvas (sharp is Node-only). Potrace runs via WASM (`esm-potrace-wasm`). This is a genuine shared core, not a port, yielding a fully client-side font builder with drag-and-drop, live preview, and TTF download.

## Key techniques

### Adaptive binarization with template-guide erasure (`capture.js`)

The binarizer builds a local background estimate by downscaling the photo 32×, blurring, then upscaling back. A pixel is ink if it's darker than both (a) a global cap (default 165 — erases light-grey printed template guides) and (b) its local background minus delta (default 40). This handles shadows, spiral binding, and uneven lighting. A morphological closing pass seals 1px threshold-flicker gaps. The approach is a clever dual condition: the cap kills the template, the delta finds the pen.

### Multi-part glyph merging (`blob-core.js`)

`mergeParts()` iteratively merges blobs that belong to a single letter, handling three cases: stacked marks (i-dot above stem), side-by-side marks (double quotes), and split strokes (disconnected M/N/K lines). It recomputes median glyph height each iteration and merges only the closest eligible pair per round — preventing a dot between two rows from joining the wrong stem. The comment in the source is instructive: "recompute each round: speck swarms (shadow noise) drag the median down until they consolidate; a stale low median blocks legitimate dot merges."

### Reading-order clustering by vertical overlap (`blob-core.js`)

`orderBlobs()` groups blobs into rows by vertical overlap rather than center-distance. Descenders (g, y) pull center-points down, which would confuse center-distance clustering into placing them in the wrong row. Vertical-overlap clustering is robust to this. Rows are then sorted left-to-right.

### Shared em-square with per-character vertical bands (`metrics.js`)

This is the craft step and the project's central architectural insight. Instead of normalizing each glyph independently (the obvious approach, which produces a ransom-note effect where every letter is a different size), every character is placed into a shared 1000-unit em square. But rather than a single band for all glyphs, each character class gets its own vertical band:

| Character class | Band (font units) |
|---|---|
| Capitals, digits | [0, 700] |
| Ascenders (b,d,f,h,k,l) | [0, 720] |
| x-height (a,c,e,m,n,o,r,s,u,v,w,x,z) | [0, 480] |
| Descenders (g,p,q,y) | [-220, 480] |
| i | [0, 660] |
| j | [-220, 660] |
| t | [0, 640] |
| Various punctuation | per-glyph |

Each glyph's ink is scaled uniformly to fill its band. This shared coordinate system produced in a single `placeGlyph()` call — `translate(pad) → scale(s, -s) → translate(lsb, top)` — is what makes the result feel like a real font. The e2e test validates this directly: `g` must descend below -100, `b` above 650, `o` at or below 500, `A` above 650.

### TrueType winding correction (`winding.js`)

Potrace emits all contours in the same direction (it assumes even-odd fill). TrueType uses NONZERO winding: outer contours clockwise, holes counter-clockwise. Without correction, counters (bowls of b, g, o, a) fill solid in real renderers. `fixWinding()` parses SVG paths into subpaths, computes signed area for winding direction and polygon containment for nesting depth, then reverses subpaths that wind the wrong way. The e2e test validates this with shoelace-formula area checks on the 'o' glyph's two contours.

### Font as direct-authored SVG-font XML (`assemble.js`)

Rather than using a font library's placement API, `buildTTF()` authors `<font>`, `<font-face>`, and `<glyph>` SVG XML elements directly with metrics exactly as `metrics.js` computed them. The XML passes through `svg2ttf` → `ttf2woff` / `wawoff2`. No icon-font "normalize" scaling anywhere in the chain. This guarantees that what `metrics.js` computed is exactly what lands in the font binary.

### Morphological weight adjustment (`trace.js`)

`adjustWeight()` dilates or erodes the binary ink mask by N pixels before tracing — a literal grow/shrink of the stroke. This produces genuinely bolder or thinner glyphs rather than stroking an outline, which would change the character of handwritten curves.

## Design decisions

### Deterministic pipeline with AI only at the edges

The project is opinionated about where AI belongs. Claude finds letters in the photo (vision), labels them, and judges the result — but it never touches the geometry. The vectorization, metrics, and font assembly are pure deterministic code. This is the opposite of most "AI-powered" tools where an LLM generates SVG paths or CSS. The result: predictable, reproducible output; no hallucinated bezier curves; and the AI layer can be swapped or removed entirely (the CLI works standalone with `--chars` or `--charset`).

### Zero system dependencies as a deliberate constraint

No FontForge, no ImageMagick, no potrace binary. Everything is npm packages. This eliminates platform-specific installation hell (FontForge is notoriously painful on Windows) and makes the tool work identically on macOS, Linux, and Windows. The trade-off: `wawoff2` (WOFF2 compression) is a native Node addon but ships prebuilt binaries. The browser demo can't do WOFF2 because the WASM port doesn't exist.

### Template-based workflow as the quality ceiling

The printable PDF template (light-grey guides erased by `--cap`) represents the project's thesis about quality: when the order of characters is known (charset = template order), no AI labeling is needed at all. The template is the best-quality path. Freeform photos work but require either human labeling (`--chars`) or AI vision (the skill). This is a pragmatic tiered approach: best quality with templates, convenience with AI.

### Skill file as infrastructure, not code

The Claude Code skill is a 141-line markdown file, not a plugin or extension. It teaches Claude how to use the existing CLI — running commands, reading contact sheets, writing label JSON, judging previews. This means the skill works with any future version of Claude Code that can follow instructions, and the skill and CLI can evolve independently. It's the same pattern as [[PAAD — Defense-in-Depth for AI-Assisted Development]] but for a single focused tool rather than a suite.

### Testing that validates architectural properties, not unit behavior

The single e2e test (`test/e2e.js`, 94 lines) validates the properties that matter: correct glyph count after merging (i-dot merged into i), per-character band placement (descenders below baseline, ascenders tall, x-height letters bounded), o-bowl winding (opposite-wound outer and hole contours), and format correctness (magic bytes). It doesn't test that `connectedComponents()` returns correct boxes — it tests that the pipeline produces a correctly structured font. This is testing the contract, not the implementation.

## Comparison notes

Unlike commercial services like Calligraphr ($8/month), which charge for servers and offer a browser-based editor, draw-your-font runs entirely locally — the user's machine does the work and the Claude Code skill serves as the editor. This inverts the cost model: zero recurring cost, zero uploads, zero credits.

Unlike [[Arabic Typography]] which deals with font engines failing to handle the complexity of Arabic script (kashida justification, contextual shaping), draw-your-font operates in the simpler Latin alphabet space where the primary challenge is getting the vertical proportions right in a shared coordinate system. The per-character band approach would need significant extension for scripts with contextual shaping.

Unlike [[Computer Use is 45x More Expensive Than Structured APIs]], which quantifies the cost gap between vision-based and API-based agent interactions, draw-your-font's Claude Code skill sits in an interesting middle ground: Claude uses vision once (to label the contact sheet), then the deterministic pipeline takes over. The vision cost is a one-time labeling step, not a per-pixel rendering cost.

The project exemplifies the pattern described in [[Layer-First Pattern — Keep Data Out of the LLM Context]]: the LLM only touches what requires judgment (which blob is which letter, does the output look right), and the raw pixel data stays in the deterministic pipeline.

---

*Sources: [[raw/draw-your-font]]*
*Last updated: 2026-07-25*
