---
topics:
  - developer-tools
---
# Xberg — Document Intelligence Engine

Xberg is the successor to Kreuzberg — a Rust document intelligence engine that extracts text, tables, structured data, and metadata from 101 file formats across 115 file extensions. Written by Na'aman Hirschfeld, it's available as a library, CLI (13 commands), REST API (`xberg serve`), and MCP server (9 tools), with Docker and Helm deployment support.

---

## Architecture

- **Front-controller pattern**: MIME detection → Extractor registry (priority-based) → Format-specific extractor → InternalDocument
- **Plugin system**: 8 plugin types (DocumentExtractor, OcrBackend, PostProcessor, Validator, EmbeddingBackend, RerankerBackend, TokenizerBackend, Renderer), all stored as `Arc<dyn Trait>` in global registries
- **Engine seam injection**: `EngineBuilder` pattern with 6 replaceable seams — CacheBackend, ProgressSink, StructuredPolicy, PresetResolver, LlmClient, ModelProvider — each defaulting to a built-in noop or core implementation
- **Feature flag architecture**: Over 100 Cargo features controlling format support, OCR backends, ML capabilities, and deployment modes; aggregate targets like `full`, `server`, `wasm-target`, `android-target`, `windows-target`, `mobile`

## Extraction Pipeline

1. MIME detection → Extractor selection by priority → Format-specific extraction → InternalDocument
2. Image OCR processing (async) → Captioning prepass (runs BEFORE main derivation to feed captions into InternalDocument)
3. Derive ExtractedDocument → Image re-encode → Base64 pass → Early post-processors → Language detection
4. **Provisional chunking** (so Middle/Late processors see chunks) → Middle post-processors → Late post-processors → Token reduction → Validators → NFC normalization
5. Discard diverged internal/formatted content (#286, #331 fixes) → LLM structured extraction → Output format → **Final chunking** (after all content mutations — #213 fix for chunk offset correctness)

## OCR Backends

- Tesseract (native C FFI + WASM), PaddleOCR (ONNX + tract), Sceptre (EasyOCR Gen2), Candle (pure-Rust transformer VLM), VLM (GPT-4V/Claude/Gemini)
- Inference backends: ONNX Runtime (default native), Sonos tract (pure-Rust, WASM/Android safe), Candle (pure-Rust transformer)

## Language Bindings

15 language bindings via the "alef" polyglot binding generator (`alef.toml`): Rust, Python, Node.js, Go, Java, C#, Ruby, PHP, Elixir, Dart, Swift, Zig, WASM, Kotlin, C FFI. The alef system scans Rust source files and emits idiomatic bindings per language.

## Code Intelligence

371 programming languages via tree-sitter, with dynamic loading on native platforms.

## Embeddings & ML

Dense (ONNX/static), Sparse (SPLADE), Late-interaction (ColBERT/MaxSim), with reranking. NER via ONNX, Candle, or LLM. Layout detection, captioning, summarization, translation, classification, QR code detection, auto-rotate, document orientation.

## Deployment Modes

- Library (embed in Rust projects)
- CLI (`xberg`) with 13 subcommands
- REST API server (`xberg serve`)
- MCP server (9 tools for agent integration)
- Docker & Helm
- WASM browser (via `xberg-wasm`)

## Key Numbers

- **v1.1.0**, Rust edition 2024, MSRV 1.91
- 20 workspace member crates
- ~1340-line feature flag configuration in the core crate alone
- Chunking runs TWICE in the pipeline (provisional early, final late after all mutations)
- Content-hash caching, per-file timeouts, parallel batch processing

## Licensing

Dual-licensed: MIT OR Apache-2.0 (unlike Kreuzberg's Elastic License 2.0 restriction).

---

*Source: [[raw/xberg]]*
*Last updated: 2026-08-07*
