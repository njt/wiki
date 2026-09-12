---
title: "pdf-inspector"
url: https://github.com/firecrawl/pdf-inspector
author: Firecrawl
date_fetched: 2026-08-14
date_published: 2026-08-13
section: "Developer Tools"
topics:
  - developer-tools
---

# pdf-inspector

Firecrawl's fast Rust library for PDF classification and text extraction. It detects whether a PDF is text-based or scanned in ~10-50ms, extracts text with position awareness, and converts to clean Markdown — all without OCR and without ML models.

## What it does

Three operations share a single document load:

- **Classification** — samples content streams for text (`Tj`/`TJ`) and image (`Do`) operators, returning one of TextBased / Scanned / ImageBased / Mixed plus a 0.0-1.0 confidence score and a per-page list of pages needing OCR.
- **Extraction** — walks PDF operators into positioned `TextItem`s (font, size, X/Y, bold/italic, underline/strikeout), handles CID fonts via ToUnicode CMaps, and detects columns and reading order.
- **Markdown** — headings via font-size tiers, lists, code blocks (monospace detection), tables (rectangle + heuristic), bold/italic, URL linking, and page-break markers.

## Why it matters

It was built for pipelines that process PDFs at scale: classify first (~20ms), extract locally (~150ms) for the ~54% of PDFs that are already text-based, and send only the scanned/image/vector remainder to expensive OCR (2-10s). On the opendataloader-bench corpus (200 PDFs, OCR disabled) it scored 0.875 overall — beating liteparse (0.873), opendataloader (0.831), pymupdf4llm (0.735), and markitdown (0.589) — while also being the fastest (0.470s for all 200 documents).

## Ship targets

One Rust core (`lopdf` as the single dependency) exposed four ways: Rust crate, Python (PyO3, abi3-py38), Node.js (napi-rs), and browser WebAssembly (wasm-bindgen, embedded CMaps, no server round-trip). Plus CLI tools (`pdf2md`, `detect-pdf`). ~127K lines of Rust, MIT licensed.
