# pdf-inspector

Firecrawl's Rust library for classifying PDFs and extracting text-to-Markdown without OCR or ML models. The core insight is a routing decision: ~54% of PDFs are already text-based and can be parsed locally in under 200ms, so detect cheaply first, extract natively for the majority, and hand only the scanned/vector/image remainder to an OCR service. A ~127K-line Rust codebase (single `lopdf` dependency) with Python, Node, and browser-WASM bindings.

---

## Architecture

A two-stage pipeline over a shared document load (`src/lib.rs` loads the PDF once and passes it to both stages):

- **Detector** (`src/detector.rs`) — parses only the xref table and page tree, then samples content streams for text (`Tj`/`TJ`) and image (`Do`) operators. Never loads all objects. Emits `PdfType` (TextBased / Scanned / ImageBased / Mixed), a confidence score, `ocr_recommended`, and `pages_needing_ocr` with per-page reason codes (`scanned`, `no_text`, `vector_text`, `suspected_garbled_text`).
- **Extractor** (`src/extractor/mod.rs`) — walks PDF operators into positioned `TextItem`s (font, size, X/Y, bold/italic/underline/strikeout) plus `PdfRect`s and `PdfLine`s. Submodules: `content_stream` (operator walker), `fonts` (widths/encodings), `xobjects` (Form XObjects with a recursion budget), `links` (hyperlinks + AcroForm), `layout` (column detection → line grouping), `reading_order`.
- **Markdown** (`src/markdown/`) — `analysis` (font stats, heading tiers) → `preprocess` (merge headings, drop caps) → `convert` (line loop + table/image insertion) → `classify` (captions, lists, code) → `postprocess`. Tables live in `src/tables/`.

Three public result types carry the whole design: `PdfProcessResult` (pdf_type, markdown, page_count, processing_time_ms, pages_needing_ocr, confidence, layout, has_encoding_issues), `TextItem`/`TextLine`, and `LayoutComplexity` (`src/types.rs`).

## Key techniques

- **Classification without full load** (`src/detector.rs:184` `detect_from_document`). Detection is a two-phase sample: Phase 1 caches per-page analysis (text ops, image count, unique chars, path ops, font changes), Phase 2 classifies from aggregate ratios. The default `ScanStrategy` is `Sample(8)` (evenly distributed pages), not "all pages" — and it detects 300+ page PDFs in milliseconds.
- **Image-dominated heuristics.** A page is `is_image_dominated` when `image_count > 10 && image_count > text_operator_count * 3`; a "template image" page (single full-page raster + sparse text) is distinguished from a text page with figures — the difference between routing an annual report to OCR and parsing it locally.
- **The Tf/Tj newspaper heuristic.** Dense multi-column newspapers (WSJ, NYT) have extractable text but garbage reading order. Detector flags them by the ratio of font-change operators (`Tf`) to text operators (`Tj`): newspapers sit at 0.02-0.06, styled contracts at 0.25-0.35. Pages with `text_ops ≥ 1500`, `font_changes ≥ 50`, and ratio `< 0.15` get routed to OCR rather than mis-parsed.
- **Geometric style detection.** PDFs have no underline/strikeout font flag, so `extractor/underline.rs` detects them geometrically — a thin drawn rule under the baseline or crossing the x-height. Sub/superscript is inferred from font-size ratio plus Y-offset (`src/types.rs` `needs_space_between`).
- **A five-deep CID-decoding ladder** (`src/tounicode.rs`). Identity-H / Type0 CID fonts are where naive extractors emit mojibake. The fallback chain is: ToUnicode CMap → subset re-mapping → TrueType cmap table (via `ttf-parser`) → encoding fallback → Adobe glyph names → CID passthrough. When a subset font's sequential remap would scramble characters, the real TrueType cmap is promoted over it.
- **Three-way table detection** (`src/tables/`). Rectangle-based (`detect_rects.rs`, union-find clustering of `re` operator rects), heuristic alignment (`detect_heuristic.rs`), and tagged-PDF structure tree (`detect_struct.rs`). Calendar layouts interpolate missing column boundaries from median spacing; `financial.rs` splits merged numeric tokens in consolidated cells.
- **Encoding-issue detection as first-class output.** `src/text_quality.rs` (`is_cid_garbage`, `is_garbage_text`, `detect_encoding_issues`) flags broken font encodings so callers fall back to OCR on mojibake — not just on scanned images.
- **WASM as a compile-time fork, not a port.** `Cargo.toml` swaps `lopdf`'s `rayon` parallel parser for `wasm_js`, embeds the ~90 CMaps via `include_dir`, and uses JS randomness for encrypted PDFs. Native is parallel; browser is deliberately single-threaded to avoid cross-origin-isolation requirements.

## Design decisions

- **Optimized for the common case.** Native-text PDFs (reports, invoices, legal, papers) get full local parsing; scanned/vector/newspaper layouts get routed onward. This is the inverse bet from [[Xberg]] or [[Dolphin]], which invest heavily in OCR/models for the hard cases and accept their cost on the easy ones.
- **No ML, one dependency.** `lopdf` is the only runtime dep. Deterministic, offline, no GPU, ~ms latency, and small enough to compile to WASM. The cost: it cannot read photographed or handwritten documents — that domain is ceded to [[Dolphin]].
- **Speed over accuracy at the margins.** Heading detection is font-size-tier heuristics (with 0.5pt clustering in `markdown/heading.rs`), not the PDF's logical structure — lossy, but the benchmark shows it still wins on tables (0.814 TEDS) and reading order (0.915 NID) against model-free peers. Tagged-PDF structure (`src/structure_tree.rs`) is read when present as a hedge.
- **Per-page OCR routing instead of all-or-nothing.** The API returns `pages_needing_ocr` and `ocr_reasons_by_page`, so a Mixed PDF can be partially extracted and partially OCR'd — a finer granularity than any of its benchmark competitors offer.

## Comparison notes

- **vs [[markitdown]]**: in Firecrawl's own benchmark markitdown scored 0.589 overall and 0.000 on headings — the generic "everything-to-Markdown" converter is weakest exactly where pdf-inspector specializes. One format done deeply beats many formats done shallowly on that format.
- **vs [[Xberg]] / [[Kreuzberg]]**: Xberg is 101 formats, 5 OCR backends, embeddings, and 15 bindings; pdf-inspector is one format, zero OCR, one dep. The polyglot engine makes OCR cheap and swappable; pdf-inspector avoids OCR for the majority entirely.
- **vs [[Dolphin]]**: both classify-then-parse, but Dolphin is a 3B-parameter VLM for photographed *and* digital docs, while pdf-inspector is deterministic Rust for digital-born PDFs. They're complementary stages: pdf-inspector as the millisecond fast path, Dolphin/OCR as the heavy path for the scanned remainder.

#tool #project #document-processing #pdf #markdown #rust

---
*Sources: [[raw/pdf-inspector]], [[summary/pdf-inspector]]*
*Last updated: 2026-08-14*
