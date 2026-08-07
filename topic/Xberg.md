# Xberg

Xberg is a Rust document intelligence engine — the successor to [[Kreuzberg]] — that extracts text, tables, metadata, embeddings, and structured data from 101 file formats across 115 file extensions. Written by Na'aman Hirschfeld, it ships as a library, CLI, REST API, MCP server, Docker container, Helm chart, and WASM module, with 15 language bindings. Licensed MIT/Apache 2.0 (unlike Kreuzberg's ELv2 restriction).

---

## Architecture

Xberg follows a **front-controller pattern**: MIME detection routes to a **priority-ordered extractor registry**, each format-specific extractor produces an `InternalDocument`, which flows through a **multi-stage post-processing pipeline** before being rendered to the requested output format.

### Core Abstractions

- **Engine**: A process-global `LazyLock<Engine>` — cloneable, wraps `Arc<EngineInner>`. The `EngineBuilder` pattern injects 6 replaceable seams (CacheBackend, ProgressSink, StructuredPolicy, PresetResolver, LlmClient, ModelProvider), each defaulting to a noop or built-in core implementation. This is classic dependency injection in Rust — every external dependency is swappable at construction time.
- **Plugin System**: 8 plugin types (DocumentExtractor, OcrBackend, PostProcessor, EmbeddingBackend, RerankerBackend, TokenizerBackend, Renderer, Validator), all stored as `Arc<dyn Trait>` in global registries with `RwLock`-protected access. Plugins can be written in Rust (native), Python (via PyO3), with Node.js planned (via napi-rs).
- **InternalDocument vs ExtractedDocument**: The double-document pattern — `InternalDocument` is the extractor's rich internal representation (element trees, page metadata, formatted content), while `ExtractedDocument` is the pipeline output (flat content, metadata, chunking). A divergence detection system (#286, #331) drops the internal representation when post-processors rewrite content, forcing renderers to use the processed text.

### Post-Processing Pipeline

The pipeline is arguably the most architecturally distinctive part of Xberg. It runs through 24 stages, but three design decisions stand out:

1. **Captioning prepass runs BEFORE the main derivation.** This is a pipelining inversion — the captioning processor derives an `ExtractedDocument` from the raw `InternalDocument`, runs its analysis (descriptions, entities, warnings), then those results are merged BACK into the `InternalDocument` before the main derivation. A `CaptioningCarryOver` struct bridges content/entities the derivation would otherwise miss.

2. **Chunking runs TWICE.** Provisional chunking runs early (after Early post-processors) so Middle and Late processors can reason about chunks. Final corrected chunking runs at the very end, after all content mutations. This is the fix for issue #213 — chunk byte offsets must index the final, post-pipeline content, not the mid-pipeline state. The second run includes NFC normalization outcomes that the first couldn't see.

3. **Divergence detection discards stale representations.** If a post-processor rewrites `result.content` but not `internal_document` (#286), the element tree is dropped so renderers fall back to processed text. If content was rewritten but formatted content wasn't (#331), the formatted content is dropped and the mismatch is logged. These are production-hardened guardrails against the silent corruption that happens when pipelines mutate documents without updating all representations.

### Binding Generation ("alef")

The `alef` system (configured via `alef.toml`) scans Rust source files and emits idiomatic bindings for 15 languages. The design is noteworthy for what it excludes — `Engine` is intentionally kept off the binding surface, and every feature-gated public function has a stub that returns `Err("requires ... feature")` so bindings always compile. Stub types mirror real types exactly, keeping JSON round-trips through serialization schema-compatible across feature configurations.

---

## Key Techniques

### Feature Flag Architecture

The core crate's `Cargo.toml` runs ~1340 lines, the majority of which is feature flag configuration. Over 100 features control format support (`pdf`, `excel`, `office`, `email`, `archives`), OCR backends (`ocr`, `paddle-ocr`, `candle-ocr`, `sceptre-ocr`), inference backends (`onnx-runtime`, `tract`, `candle`), ML capabilities (`embeddings`, `sparse-embeddings`, `late-interaction`, `reranker`, `ner`, `captioning`, `summarization`, `translation`, `classification`), and deployment modes. Aggregate targets (`full`, `full-no-heic`, `server`, `wasm-target`, `android-target`, `windows-target`, `mobile`) compose features for specific platforms.

This is extreme granularity taken to its logical conclusion — you can compile exactly what you need and nothing more, which matters when targeting WASM (where binary size is critical) or mobile (where every dependency adds build complexity).

### OCR Architecture

Five OCR backends spanning the spectrum from classical to frontier:
- **Tesseract** — native C FFI for desktop, WASM for browser
- **PaddleOCR** — ONNX Runtime (native) or Sonos tract (pure-Rust, WASM/Android safe)
- **Sceptre** — EasyOCR Gen2 with ONNX/tract backends
- **Candle** — pure-Rust transformer VLM for OCR (zero system dependencies)
- **VLM** — GPT-4V/Claude/Gemini for highest quality at API cost

The backend abstraction means you can toggle from offline Tesseract to cloud VLM with a config switch — no code changes.

### Embedding Strategy

Three embedding types with distinct retrieval paradigms:
- **Dense** (ONNX/static) — standard vector similarity
- **Sparse** (SPLADE) — term-weight vectors for lexical-semantic hybrid
- **Late-interaction** (ColBERT/MaxSim) — per-token embeddings for fine-grained matching

Plus reranking via cross-encoder models. The architecture supports embedding generation as a separate phase from extraction, with `embed_texts()`, `rerank()`, `embed_sparse()`, and `embed_multi_vector()` as top-level API functions.

### Deployment Versatility

Six deployment modes from a single codebase: embeddable Rust library, CLI binary, REST API server, MCP server (9 tools), Docker/Helm containers, WASM browser module. The MCP server is particularly relevant for agent workflows — an AI agent can use Xberg as a tool to ingest any document format and get structured text plus embeddings back.

---

## Design Decisions

### MIT/Apache 2.0 vs ELv2

The most consequential decision relative to Kreuzberg: Xberg is dual-licensed MIT OR Apache 2.0, while Kreuzberg used Elastic License 2.0. ELv2 prohibited offering Kreuzberg as a managed service. The MIT/Apache 2.0 dual license removes that restriction — you can build commercial products on Xberg, embed it in SaaS, or wrap it in a paid API. This is a bet on adoption over monetization.

### Double Chunking

Running chunking twice (provisional early, final late) is a correctness-over-simplicity trade-off. The alternative — running chunking only at the end — would mean Middle and Late processors can't reason about document structure. The compromise — final chunking runs after all mutations so byte offsets are correct — adds complexity but eliminates a class of off-by-N bugs that plague document pipelines.

### Stub Pattern for Feature-Gated APIs

Every feature-gated public function has a compile-time stub that returns a validation error. This is more code but guarantees bindings always compile regardless of feature configuration. The alternative — conditional compilation in binding generators — would be a maintenance nightmare across 15 languages.

### Monorepo with Granular Feature Flags

The workspace contains 20 crates, but the core `xberg` crate is where nearly all logic lives — feature flags toggle sub-modules rather than separate crates for each capability. This is a monolith architecture gated at compile time, not a microservices architecture. For a library that needs to be embeddable everywhere (including WASM), this is pragmatically correct — dynamic plugin loading is a non-starter in many target environments.

### Content-Hash Caching

Each document is hashed and cached, so re-processing the same file returns instantly. This is table-stakes for a library, but the implementation detail matters: the cache is an injected seam (CacheBackend trait), so production deployments can swap in Redis or S3-backed caches while development uses the in-memory default.

---

## Comparison Notes

### vs Kreuzberg

Xberg IS Kreuzberg's successor — same author (Na'aman Hirschfeld), same core mission. The differences: MIT/Apache 2.0 licensing (vs ELv2), 101 formats (vs 97), 15 bindings (vs 14), a significantly more sophisticated post-processing pipeline with double chunking and divergence detection, five OCR backends (vs three), three embedding types, and NER/summarization/translation/classification capabilities that Kreuzberg didn't have. The CLI grew from a few commands to 13, and the MCP server went from 1 tool to 9. Xberg is Kreuzberg grown up and opened up.

### vs markitdown

[[markitdown]] converts office documents to Markdown for LLM pipelines. Xberg converts 101 formats to structured output (Markdown, plain text, JSON, TOON) WITH embeddings, OCR, chunking, NER, and LLM-mediated enrichment. markitdown is a single-purpose converter; Xberg is a multi-stage intelligence pipeline. If you need "get text from this doc into my LLM," both work. If you need "extract, embed, chunk, classify, summarize, and index these 10,000 documents," only Xberg does the full stack.

### vs Dolphin

[[Dolphin]] (ByteDance) is a universal document parsing model — a single neural network that handles digital and photographed documents. Xberg takes the opposite approach: format-specific extractors (PDF, Office, email, archive) plus pluggable OCR backends. Dolphin bets on model capability; Xberg bets on pipeline composition. The trade-off is accuracy vs flexibility — Dolphin may do better on its target formats, but Xberg can add a new format by writing a Rust extractor rather than retraining a model.

### Deployment Breadth

Few document processing tools match Xberg's deployment versatility. Most are either a library OR a CLI OR an API. Xberg is all six. The WASM target is particularly notable — running document extraction in a browser with no server round-trip opens use cases (client-side sensitive document processing, offline-first apps) that server-only tools can't address. The MCP server bridges to the agent ecosystem, making Xberg directly usable from Claude Code, Codex, and other MCP-speaking tools.

---

*Sources: [[raw/xberg]]*
*Last updated: 2026-08-07*
