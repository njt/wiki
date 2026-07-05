---
url: https://github.com/chenglou/pretext
title: "Pretext — Fast, accurate & comprehensive text measurement & layout"
author: Cheng Lou
date_fetched: 2026-05-14
date_published: unknown
---

# Pretext

Fast, accurate & comprehensive text measurement & layout. Pure JavaScript/TypeScript library for multiline text measurement and layout.

46.9k stars, MIT license. TypeScript 91.2%, HTML 8.8%.

## Overview

Pretext side-steps the need for DOM measurements (e.g. `getBoundingClientRect`, `offsetHeight`), which trigger layout reflow. Instead, it implements custom text measurement logic using the browser's font engine, performing pure arithmetic over cached widths.

Supports multilingual text (Latin, CJK, Arabic, emoji).

## Installation

```
npm install @chenglou/pretext
```

## Use Case 1: Measuring paragraph height without DOM interaction

```javascript
const prepared = prepare('AGI 春天到了. بدأت الرحلة 🚀‎', '16px Inter')
const { height, lineCount } = layout(prepared, 320, 20)
```

- `prepare()`: performs text normalization, segmentation, and measurement caching
- `layout()`: executes pure arithmetic over cached widths for height determination

## Use Case 2: Manual paragraph line layout

```javascript
const prepared = prepareWithSegments('Text here', '18px "Helvetica Neue"')
const { lines } = layoutWithLines(prepared, 320, 26)
```

For canvas, SVG, or custom rendering with granular control over individual lines.

### APIs

- `layoutWithLines()`: returns individual line objects
- `walkLineRanges()`: iterates through lines without string allocation
- `layoutNextLineRange()`: enables variable-width layout (text flowing around obstacles)
- `materializeLineRange()`: converts ranges to full lines

## Rich Text Inline Support

Helper module at `@chenglou/pretext/rich-inline` handles inline text including boundary spaces, with features like atomic items (`break: 'never'`) and caller-owned width adjustments.

## Configuration Options

Both preparation functions accept options:
- `whiteSpace`: 'normal' or 'pre-wrap'
- `wordBreak`: 'normal' or 'keep-all'
- `letterSpacing`: numeric pixel value

## Additional Functions

- `clearCache()`: releases accumulated font caches
- `setLocale()`: configures locale for text processing
- `measureLineStats()`: provides line counts and widths
- `measureNaturalWidth()`: returns widest forced line width

## Limitations

Targets "the common text setup" and doesn't support advanced CSS features like `font-optical-sizing` or `font-feature-settings`. Requires `Intl.Segmenter` and Canvas 2D text measurement support.

## Links

- Live demos: chenglou.me/pretext/
- Additional demos: somnai-dreams.github.io/pretext-demos/
- Development guide: DEVELOPMENT.md
