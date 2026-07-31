---
url: https://github.com/azu/har-extractor
title: har-extractor
author: azu
date_fetched: 2026-08-01
date_published: 2018-06-20
---

# har-extractor — Source Analysis

## Project Overview

A CLI tool (and library) that extracts response bodies from HAR (HTTP Archive) files into a directory tree. The core extract function is 63 lines of TypeScript; the CLI wrapper adds another 60 lines of JavaScript.

- **Repository**: https://github.com/azu/har-extractor
- **Author**: azu (azu_re on Twitter)
- **License**: MIT
- **Language**: TypeScript (compiled to ES5/CommonJS)
- **Version**: 1.1.2 (2022-02-24)
- **Dependencies**: filenamify, humanize-url, make-dir, meow (CLI only)
- **Dev Dependencies**: @types/har-format, mocha, ts-node, prettier

## File Structure

```
src/har-extractor.ts    — core library (63 lines)
bin/cmd.js              — CLI entry point (60 lines)
test/har-extractor-test.ts — 4 test cases with 3 HAR fixtures
test/fixtures/           — en.wikipedia.org.har, hatebupwa.netlify.com.har, seventow.har
```

## Core Logic: `extract()` function

The `extract` function (`src/har-extractor.ts:46-63`) is a single-pass synchronous pipeline:

1. Iterate `har.log.entries` — each entry represents one HTTP request/response pair
2. Call `getEntryContentAsBuffer(entry)` to decode `response.content.text` into a Buffer
3. Call `convertEntryAsFilePathFormat(entry)` to derive a filesystem path from the request URL
4. `makeDir.sync()` the parent directories, then `fs.writeFileSync()` the buffer

Options control verbosity (`--verbose`), dry-run mode (`--dry-run`), and query string stripping (`--remove-query-string`).

### `getEntryContentAsBuffer()` (lines 8-19)

Handles the two encoding formats HAR uses for inline response bodies:

- **`encoding: "base64"`**: `Buffer.from(text, "base64")` — decodes binary content stored as base64 strings
- **No encoding / default**: `Buffer.from(text)` — passes text content through directly

Entries with no `content.text` return `undefined` and are silently skipped (binary content stored in external files isn't handled).

### `convertEntryAsFilePathFormat()` (lines 21-37)

The clever URL-to-path conversion:

1. **`humanizeUrl(url)`**: Strips the protocol prefix, `www.`, and trailing slashes. `https://en.wikipedia.org/wiki/.har` → `en.wikipedia.org/wiki/.har`

2. **Optional query string removal**: If `removeQueryString` is true, splits on `?` and takes only the path portion. This is useful for URLs with cache-busting params.

3. **`filenamify(pathname, {maxLength: 255})`**: Sanitizes each path segment for filesystem safety — replaces reserved characters (`<`, `>`, `:`, `"`, `/`, `\`, `|`, `?`, `*`), handles trailing dots/spaces, and truncates to 255 characters.

4. **The `index.html` insertion heuristic** (lines 28-35): When a URL path doesn't contain `.html` but the response MIME type is `text/html`, appends `/index.html` to the path. This turns directory-like URLs into proper filesystem paths — so `https://en.wikipedia.org/wiki/.har` becomes `en.wikipedia.org/wiki/.har/index.html`, which a browser can open directly.

## CLI Entry Point (`bin/cmd.js`)

Uses the `meow` library for argument parsing with four flags: `--output` (required, directory path), `--remove-query-string` (`-r`, boolean), `--dry-run` (boolean), and `--verbose` (boolean, default `true`).

The CLI reads the HAR file with `JSON.parse(fs.readFileSync(...))`, calls `extract()`, and on error prints the error message and shows help. Notably, the `output` flag has no default — if omitted, `outputDir` is `undefined` and the tool will fail when trying to join paths.

## Tests

Four test cases in `test/har-extractor-test.ts`:

1. **Wikipedia HAR extraction**: Extracts the en.wikipedia.org HAR fixture and verifies output files are created
2. **GitHub issue #6 regression**: Extracts the `seventow.har` fixture (a specific bug report case)
3. **Hatebupwa HAR extraction**: Extracts the hatebupwa.netlify.com.har fixture
4. **Dry-run mode**: Verifies that `--dry-run` produces zero output files

Tests clean up after each run via `del([outputDir])` in `afterEach`.

## What the Tool Does NOT Do

- **No content-encoding decompression**: HAR entries may have `gzip`, `deflate`, or `br` content-encoding, but the tool assumes the HAR generator already decompressed the body. This is typical for browser-generated HARs (Chrome DevTools decompresses before saving).
- **No streaming/incremental processing**: The entire HAR JSON is loaded into memory, and all entries are written synchronously. For large HAR files (multi-GB from long recording sessions), this could be a problem.
- **No incremental/partial extraction**: Every entry is extracted every time. No caching, no skip-if-exists logic.
- **No error recovery per entry**: If any entry fails, the entire operation stops (sync code, no try/catch per iteration).
- **No concurrent writes**: Everything is synchronous — one entry at a time, in order.

## Design Trade-offs

**Simplicity over robustness**: The entire core is 63 lines. You can read and understand the whole tool in under a minute. The trade-off is no error recovery, no streaming, and no partial extraction — but for the target use case (browser-generated HAR files from a single page load, typically a few MB), these aren't problems.

**URL-as-path over content-addressing**: The tool uses the request URL as the filesystem path, preserving the site's directory structure. An alternative would be content-addressing (hash the response body) for deduplication, but that loses the navigable URL tree. The URL-as-path approach makes the output directory browsable and mirrors the original site structure.

**The index.html heuristic is opinionated**: Not every URL without `.html` in the path is a "directory" that needs `index.html` — API endpoints returning JSON with `text/html` MIME type (misconfigured) would get incorrectly renamed. But for the common case of browser DevTools HAR captures, this heuristic is exactly right: `/wiki/.har` returning HTML should produce a filesystem path you can open in a browser.

**Sync I/O is the right call for this size**: At 63 lines and targeting files that fit in memory, async I/O would add complexity without benefit. The tool is meant to be used on HAR files from single page recordings, not terabyte-scale archives.

## npm Package Architecture

- Main entry: `lib/har-extractor.js` (compiled TypeScript)
- CLI binary: `./bin/cmd.js` (requires the compiled library)
- TypeScript declarations: `lib/har-extractor.d.ts`
- Build: `tsc` with `strict: true`, `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`
- Published files: `bin/`, `lib/`, `src/` (ships source for source-map debugging)
