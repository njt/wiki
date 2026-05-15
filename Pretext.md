# Pretext

A pure JavaScript/TypeScript library that measures and lays out multiline text without touching the DOM. Instead of triggering expensive layout reflows via `getBoundingClientRect` or `offsetHeight`, Pretext caches font metrics and does the math itself — giving you pixel-accurate paragraph heights, line breaks, and line-by-line layout using only the browser's Canvas 2D text measurement API. 46.9k GitHub stars, MIT licensed.

---

## Key Quotes

> "Side-step the need for DOM measurements (e.g. `getBoundingClientRect`, `offsetHeight`), which trigger layout reflow."

The core pitch in one sentence: avoid reflow by reimplementing the text layout algorithm in userspace.

> `prepare()` does text analysis and measurement; `layout()` is "pure arithmetic over cached widths."

The two-phase design — expensive work once, cheap work many times — is the architectural insight that makes it practical.

## Key Themes

#tool #concept #pattern

- **DOM reflow avoidance** — The library exists because DOM text measurement is synchronous and forces layout. Every `getBoundingClientRect` call is a hidden performance cliff.
- **Userspace text layout** — Reimplementing what the browser does, but in a way the application controls. Same family as game engines that roll their own text rendering.
- **Multilingual text segmentation** — Handles Latin, CJK, Arabic, and emoji correctly via `Intl.Segmenter`. This is where most naive text measurement breaks.
- **Canvas/SVG rendering pipeline** — The `layoutWithLines()` and `walkLineRanges()` APIs are clearly designed for people building custom renderers — collaborative editors, design tools, games.
- **Variable-width text flow** — `layoutNextLineRange()` supports text flowing around obstacles, which is a genuinely hard layout problem that CSS only handles with `shape-outside`.

## Critical Analysis

**What's strong:** The API design is clean — two entry points (`prepare`/`layout` for simple measurement, `prepareWithSegments`/`layoutWithLines` for full control) that don't leak abstraction. The two-phase prepare/layout split means you can cache the expensive measurement step and re-layout cheaply when only the container width changes (e.g., window resize). Multilingual support via `Intl.Segmenter` is the right call — most JS text measurement libraries silently break on CJK or RTL text.

**What's missing:** No mention of how it handles web fonts that load asynchronously — if you `prepare()` before the font is ready, your cached metrics are wrong. The library targets "the common text setup" and explicitly excludes `font-optical-sizing` and `font-feature-settings`, which limits its use for typographically ambitious applications. No server-side story — `Intl.Segmenter` and Canvas 2D are browser APIs, so this is client-only.

**Why it matters:** This is the kind of library that collaborative editing tools (Figma, Google Docs, Notion) need internally. If you're building anything that renders text outside the normal DOM flow — a canvas-based editor, a custom text renderer, a design tool — you either build this yourself or use something like Pretext. The 46.9k stars suggest a lot of people are building exactly that. Cheng Lou (creator of Reason, ReasonML, and React Motion) has a track record of well-designed developer tools.

**Connections:** The variable-width text flow feature is related to the kind of layout problems that design tools like Figma solve. The caching architecture (expensive prepare, cheap layout) echoes the prepare/execute split in database query engines. The "avoid the DOM" philosophy connects to the broader trend of moving rendering out of the browser's layout engine — same impulse behind [[Browser Use]] for automation, but here applied to text measurement.

---

*Sources: [[raw/pretext]]*
*Last updated: 2026-05-14*
