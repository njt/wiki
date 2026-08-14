# markitdown

Microsoft's utility for converting office documents to Markdown. PDF, PowerPoint, Word, Excel, images (with OCR), audio (with transcription), HTML, CSV, JSON, XML, ZIP, YouTube URLs, EPub -- all to Markdown. Designed for LLM pipelines, not human reading.

---

## Key Quotes

> "Markdown is extremely close to plain text, with minimal markup or formatting, but still provides a way to represent important document structure."

> Markdown conventions are "highly token-efficient."

## Key Themes

#tool #document-conversion #markdown #llm-pipeline

The argument for Markdown as the universal intermediate format is compelling: LLMs already understand it natively (they generate it unprompted), it preserves document structure without heavy markup overhead, and it's token-efficient. When you're feeding documents into an LLM context window, every saved token matters.

The breadth of format support is the real value. Most conversion tools handle a few formats well. markitdown handles nearly everything, which means you can build a single pipeline that ingests any document type. The plugin system allows extending to new formats.

## Critical Analysis

The key caveat: this prioritizes structure preservation over human-friendly presentation. The output is meant to be consumed by machines, not read by people. If you want beautiful Markdown from a Word doc, look elsewhere. If you want reliable structure extraction for an LLM to reason over, this is the tool.

For PDF specifically markitdown is the weak link in the family: Firecrawl's opendataloader benchmark scored it 0.589 overall (0.000 on headings, 0.273 on tables) against [[pdf-inspector]]'s 0.875 — a specialized Rust PDF parser that classifies and extracts natively in under 200ms.

The security warning (performs I/O with current process privileges) matters in production. If you're building a document ingestion pipeline, sandbox this. Untrusted documents are a classic attack vector.

For a full document intelligence pipeline (extraction + OCR + embeddings + chunking + NER + classification across 101 formats), see [[Xberg]] — a Rust engine that subsumes markitdown's conversion role within a much larger extraction and enrichment pipeline. Compare with [[docmason]], which takes the opposite approach -- preserving original document structure and enforcing source boundaries for citation. markitdown converts to a flat format; docmason maintains the relational structure. Different problems, complementary tools.

See also [[graphify]] for turning codebases (rather than documents) into structured knowledge.

---
*Sources: [[summary/markitdown]]*
*Last updated: 2026-05-14*
