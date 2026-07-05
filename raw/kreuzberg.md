---
url: https://github.com/kreuzberg-dev/kreuzberg
date_fetched: 2026-07-05
backfilled: true
---

One Rust engine — 96 file formats, 306 programming languages, **native bindings for 15 languages**, dual model runtimes, 6 output formats, OCR from any backend, embeddings, structured LLM extraction, token reduction, and more.


Xberg is the next iteration of Kreuzberg.Same document-intelligence engine, rebuilt and rebranded under a fresh v1 line.

**Feed documents → get clean text, tables, metadata, transcripts, code intelligence · Run it library, CLI, REST API, or MCP server · No GPU needed · Stream multi-GB files · Cache results.**

Documents · Images · Spreadsheets · Email · Archives · Code · Audio · Video

Quick start · What you get · Capabilities · CLI · Docs

*Feed any document—get structured text. Extract, batch, stream, or crawl.*

Xberg is a full content-intelligence engine. One Rust core with fast, accurate extraction from 96 file formats and 306 programming languages. Language bindings for Rust, Python, Node.js, Go, Java, C#, Ruby, PHP, Elixir, Dart, Swift, Zig, WASM, Kotlin, and C FFI. Use it as a library, CLI tool, REST API, or MCP server.

| What it does | How | 
|---|---|
| Extract from 96 formats | PDFs, Office, images, HTML, email, archives, scientific publications, and code — intelligent MIME detection, streaming for large files. | 
| 6 output formats | Plain text, Markdown, Djot, HTML, JSON tree structure, or Structured (JSON with OCR metadata and bounding boxes). | 
| Code intelligence | Functions, classes, imports, symbols, docstrings from 306 programming languages. Syntax-aware chunking for RAG pipelines. | 
| Crawl & recurse | Follow URLs, extract documents from within documents (nested archives, embedded PDFs). Auto/Document/Crawl modes. | 
| OCR on demand | Tesseract, PaddleOCR, Candle, or VLM backends — fallback chains, extensible via plugins. Confidence scores. Language auto-detection. | 
| Transcription | Whisper ONNX for audio/video tracks (MP3, M4A, WAV, WebM, MP4). | 
| Embeddings & search | Local (ONNX models) or provider-hosted (OpenAI, Anthropic, Google, 143 providers via liter-llm). Reranking. | 
| Structured outputs | LLM-powered extraction — local (Ollama, LM Studio, vLLM) or remote (OpenAI, Anthropic, Google). | 
| Enrichment | NER, redaction, summarization, translation, QR code detection, page classification, keyword extraction (YAKE/RAKE), language detection, layout detection, table extraction, token reduction (TOON). | 
| Batch & parallel | Process 100s of documents in parallel. Per-file timeouts. Configurable batch concurrency ( `max_concurrent_extractions`). | 
| Caching | Content-hash cache keys — skip re-extraction when the file and config are unchanged. | 
| Deployment | Library, CLI (12 commands), REST API ( `xberg serve`), MCP server (9 tools, 3 prompts, 4 resources), Docker. | 

*The CLI: 12 commands for extraction, caching, serving, and MCP.*

*OCR with confidence scores and bounding boxes. Switch backends without code changes.*

*Web crawl: fetch a page, follow links, extract all documents recursively.*

*MCP server: AI agents extract documents, detect formats, warm models, manage cache.*

*REST API: stream large files, get JSON or Markdown, one endpoint for all formats.*

**Java**

Available on Maven Central as `io.xberg:xberg`. See Java README for the dependency snippet.

**Elixir**

Add `{:xberg, "~> 1.0"}` to your `mix.exs` dependencies. See Elixir README for full documentation.

**Kotlin (Android)**

Available on Maven Central as `io.xberg:xberg-android`. See Kotlin README for the dependency snippet.

**Swift**

Add via Swift Package Manager. See Swift README for full documentation.

**Zig**

Add via `zig fetch`. See Zig README for full documentation.

**C/C++ (FFI)**

Build from source as part of this workspace. See C (FFI) README for full documentation.

**CLI Tool**

`brew install xberg-io/tap/xberg`12 commands: `extract`, `batch`, `detect`, `formats`, `version`, `cache` (stats/clear/manifest/warm), `serve`, `mcp`, `api`, `embed`, `chunk`, `completions`.

See CLI usage guide for detailed documentation.

**Docker**

`docker pull ghcr.io/xberg-io/xberg:latest`Run in API, CLI, or MCP modes. See Docker guide for examples.

**REST API Server**

`xberg serve --host 0.0.0.0 --port 8000`One POST endpoint handles all formats. Returns JSON or Markdown. Stream large files. See API server guide.

**MCP Server**

`xberg mcp --transport stdio`9 tools (extract, extract_batch, detect_mime_type, cache_stats, list_formats, cache_clear, get_version, cache_manifest, cache_warm). 3 prompts (extract_document, extract_with_ocr, semantic_search). 4 resources (formats, models, OCR languages, embedding presets).

Add to Claude Desktop or Cursor:

```
{
  "mcpServers": {
    "xberg": { "command": "xberg", "args": ["mcp"] }
  }
}
```
Install the Xberg plugin from `xberg-io/plugins`. Ships extraction APIs, OCR backends, configuration, and language conventions.

**Claude Code**

```
/plugin marketplace add xberg-io/plugins
/plugin install xberg@xberg
```
**Codex CLI**

```
/plugins add https://github.com/xberg-io/plugins
```
Search for `xberg` and select **Install Plugin**.

**Cursor**

Settings → Plugins → Add from URL → `https://github.com/xberg-io/plugins`, then select **xberg**.

**Gemini CLI**

```
gemini extensions install https://github.com/xberg-io/plugins
```
**Factory Droid**

```
droid plugin marketplace add https://github.com/xberg-io/plugins
droid plugin install xberg@xberg
```
**GitHub Copilot CLI**

```
copilot plugin marketplace add https://github.com/xberg-io/plugins
copilot plugin install xberg@xberg
```
**opencode**

Add to `opencode.json`:

```
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@xberg-io/opencode-xberg"]
}
```
Extract text from a document:

```
use xberg::{extract, ExtractInput, ExtractionConfig};
#[tokio::main]
async fn main() -> xberg::Result<()> {
    let config = ExtractionConfig::default();
    let output = extract(
        ExtractInput::from_uri("document.pdf"),
        &config
    ).await?;
    println!("{}", output.results[0].content);
    Ok(())
}
```
Common use cases — see Quick start guide for language-specific examples, OCR, batch processing, and API configuration.

**Full feature list**

96 file formats across 8 major categories with intelligent format detection and comprehensive metadata extraction.

| Category | Formats | Capabilities | 
|---|---|---|
| Word Processing | `.docx`,`.docm`,`.doc`,`.dotx`,`.dotm`,`.dot`,`.odt`,`.pages` | Full text, tables, images, metadata, styles | 
| Spreadsheets | `.xlsx`,`.xlsm`,`.xlsb`,`.xls`,`.xla`,`.xlam`,`.xltm`,`.xltx`,`.xlt`,`.ods`,`.numbers` | Sheet data, formulas, cell metadata, charts | 
| Presentations | `.pptx`,`.pptm`,`.ppt`,`.ppsx`,`.potx`,`.potm`,`.pot`,`.key` | Slides, speaker notes, images, metadata | 
| PDF | `.pdf` | Text, tables, images, metadata, OCR support | 
| eBooks | `.epub`,`.fb2` | Chapters, metadata, embedded resources | 
| Database | `.dbf` | Table data extraction, field type support | 
| Hangul | `.hwp`,`.hwpx` | Korean document format, text extraction | 

| Category | Formats | Features | 
|---|---|---|
| Raster | `.png`,`.jpg`,`.jpeg`,`.gif`,`.webp`,`.bmp`,`.tiff`,`.tif` | OCR, table detection, EXIF metadata, dimensions, color space | 
| Advanced | `.jp2`,`.jpx`,`.jpm`,`.mj2`,`.jbig2`,`.jb2`,`.pnm`,`.pbm`,`.pgm`,`.ppm` | OCR via pure-Rust JPEG2000 decoder, JBIG2 support, table detection | 
| HEIC family | `.heic`,`.heics`,`.heif`,`.avif`,`.avcs` | EXIF metadata, optional pixel decoding | 
| Vector | `.svg` | DOM parsing, embedded text, graphics metadata | 

| Category | Formats | Features | 
|---|---|---|
| Audio | `.mp3`,`.mpga`,`.m4a`,`.wav`,`.webm` | Whisper transcription | 
| Video audio track | `.mp4`,`.mpeg`,`.webm` | Audio-track transcription only | 

| Category | Formats | Features | 
|---|---|---|
| Markup | `.html`,`.htm`,`.xhtml`,`.xml`,`.svg` | DOM parsing, metadata (Open Graph, Twitter Card), link extraction | 
| Structured Data | `.json`,`.yaml`,`.yml`,`.toml`,`.csv`,`.tsv` | Schema detection, nested structures, validation | 
| Text & Markdown | `.txt`,`.md`,`.markdown`,`.djot`,`.mdx`,`.rst`,`.org`,`.rtf` | CommonMark, GFM, Djot, MDX, reStructuredText, Org Mode | 

| Category | Formats | Features | 
|---|---|---|
| Email | `.eml`,`.msg`,`.pst` | Headers, body (HTML/plain), attachments, threading | 
| Archives | `.zip`,`.tar`,`.tgz`,`.gz`,`.7z` | File listing, nested archives, metadata, recursive extraction | 

| Category | Formats | Features | 
|---|---|---|
| Citations | `.bib`,`.ris`,`.nbib`,`.enw` | Structured parsing: RIS, PubMed/MEDLINE, EndNote XML, BibTeX/BibLaTeX | 
| Scientific | `.tex`,`.latex`,`.typ`,`.typst`,`.jats`,`.ipynb` | LaTeX, Typst, Jupyter notebooks, PubMed JATS | 
| Publishing | `.fb2`,`.docbook`,`.dbk`,`.docbook4`,`.docbook5`,`.opml` | FictionBook, DocBook XML, OPML outlines | 

Extract structure from 306 programming languages via tree-sitter:

| Feature | Description | 
|---|---|
| Structure Extraction | Functions, classes, methods, structs, interfaces, enums | 
| Import/Export Analysis | Module dependencies, re-exports, wildcard imports | 
| Symbol Extraction | Variables, constants, type aliases, properties | 
| Docstring Parsing | Google, NumPy, Sphinx, JSDoc, RustDoc, and 10+ formats | 
| Syntax-Aware Chunking | Split code by semantic boundaries for RAG pipelines | 
| Diagnostics | Parse errors with line/column positions | 

Powered by tree-sitter-language-pack.

| Format | Use case | Example | 
|---|---|---|
| Plain | Raw text, no markup | `"Chapter 1\nIntroduction"` | 
| Markdown | Readable, structured, RAG-friendly | `"# Chapter 1\n## Introduction"` | 
| Djot | Modern lightweight markup | Similar to Markdown but stricter | 
| HTML | Styled, browser-ready | `<h1>Chapter 1</h1>` | 
| JSON | Machine-readable tree structure | Hierarchical sections with heading levels | 
| Structured | OCR metadata, bounding boxes | JSON with `elements[]`containing`{text, bbox, confidence}` | 

| Mode | Command | Transport | Use case | 
|---|---|---|---|
| Library | `xberg::extract()` | Async functions | Embed in your application | 
| CLI | `xberg extract document.pdf` | 12 commands | Scripts, batch jobs, CI/CD | 
| REST API | `xberg serve` | HTTP POST | Microservice, serverless deployment | 
| MCP Server | `xberg mcp` | stdio or HTTP | Claude, Cursor, IDE agents | 
| Docker | `docker run ghcr.io/xberg-io/xberg` | All modes | Container deployment | 

- **Tesseract**— Native C FFI (Linux/macOS/Windows) and WASM (browser)
- **PaddleOCR**— ONNX Runtime, mobile-optimized models
- **Candle**— Pure Rust, CPU-only, lightweight
- **VLM**— GPT-4 Vision, Claude Vision, Gemini Vision, or 143 providers via liter-llm

Fallback chains. Extensible via plugin system.

**Local (ONNX Runtime):**

- Preset models: fast, balanced (default), quality, multilingual
- Dimensions: 384, 768, 1024

**Provider-hosted:**

- OpenAI, Anthropic, Google, Hugging Face, Mistral, Cohere, and 143 providers total
- Via liter-llm integration

**Reranking:**

- Local ONNX rerankers (cross-encoder models)
- Provider-hosted: Cohere Rerank, others

Local engines: Ollama, LM Studio, vLLM

Remote: OpenAI, Anthropic, Google, Mistral, Cohere, and 143 providers via liter-llm

Schema validation. Temperature, top-p, frequency penalty tuning.

- **NER**— GLiNER or LLM-based entity recognition
- **Redaction**— Mask PII (phone, email, SSN, credit card, addresses)
- **Summarization**— Document and section summaries via LLM
- **Translation**— Multi-language via LLM
- **Page Classification**— Tag document pages (cover, toc, content, etc.)
- **QR Code Detection**— Extract and decode QR codes from images
- **Keyword Extraction**— YAKE or RAKE algorithms
- **Language Detection**— Detect document language
- **Layout Detection**— RT-DETR + TATR models for document structure
- **Table Extraction**— Cell-level structure and content
- **Token Reduction**— TOON wire format (~30–50% fewer tokens than JSON)

**All 12 commands**

| Command | Subcommands | Purpose | 
|---|---|---|
| `extract` | — | Extract text from a single document (path, URL, or stdin) | 
| `batch` | — | Extract from multiple documents in parallel | 
| `detect` | — | Identify MIME type of a file | 
| `formats` | — | List all 96 supported formats and MIME types | 
| `version` | — | Show Xberg version | 
| `cache` | `stats`,`clear`,`manifest`,`warm` | Manage extraction cache and models | 
| `serve` | — | Start REST API server (default: http://127.0.0.1:8000) | 
| `mcp` | — | Start MCP server (stdio or HTTP transport) | 
| `api` | `schema` | Output OpenAPI 3.1 specification | 
| `embed` | — | Generate embeddings for text (local or provider-hosted) | 
| `chunk` | — | Split text into chunks (text, markdown, YAML, or semantic) | 
| `completions` | — | Generate shell completion scripts | 

Run `xberg --help` or `xberg <command> --help` for detailed options.

Full guides, API references for every binding, format reference, and configuration docs live at **xberg.io**.

- Getting Started
- Quick Start
- Guides
- API Reference
- Format Reference
- Live Demo (browser, WASM)

Contributions are welcome! See CONTRIBUTING.md for guidelines.

Join our Discord community for questions and discussion.

Xberg is one of six open-source projects from Kreuzberg, Inc.:

- Xberg — document intelligence: text, tables, metadata from 91+ formats with optional OCR.
- Xberg Enterprise — managed extraction API with SDKs, dashboards, and observability.
- crawlberg — web crawling and scraping with HTML→Markdown and headless-Chrome fallback.
- html-to-markdown — fast, lossless HTML→Markdown engine.
- liter-llm — universal LLM API client with native bindings for 14 languages and 143 providers.
- tree-sitter-language-pack — tree-sitter grammars and code-intelligence primitives.
- alef — the polyglot binding generator that produces every per-language binding across the 5 polyglot repos.

MIT License (MIT) — see LICENSE for details.
