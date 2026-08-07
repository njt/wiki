# Kreuzberg

**Superseded by [[Xberg]]** (2026, same author). Xberg relicenses from ELv2 to MIT/Apache 2.0, adds 4 more formats, a fifth OCR backend, three embedding types (dense/sparse/late-interaction), NER/summarization/translation/classification, double-chunking pipeline with divergence detection, and grows from 1 MCP tool to 9. The architecture and philosophy carry forward — the license is the headline change for anyone building products on it.

A polyglot document intelligence framework with a Rust core. Extracts text, metadata, and structured information from PDFs, Office documents, images, and 97+ formats. Available for basically every language you can think of, plus CLI, REST API, and MCP server.

---

## Key Themes

#tool #document-processing #rust #mcp

Kreuzberg is the Swiss Army knife of document extraction. The breadth is staggering: 97+ file formats, 14+ language bindings, 306 programming languages for code intelligence via tree-sitter, multiple OCR backends including VLM-based OCR through GPT-4o/Claude/Gemini.

The MCP server deployment option is what makes this interesting for agent workflows. An AI agent can use Kreuzberg as a tool to ingest any document format -- PDFs, emails, archives, academic papers -- and get structured text back. This is the plumbing that makes [[Agentic Coding]] practical for document-heavy workflows.

The TOON wire format claiming 30-50% fewer tokens than JSON for RAG pipelines is a bold optimization. If your pipeline processes thousands of documents, that token savings compounds fast.

## Critical Analysis

The Elastic License 2.0 is a yellow flag. ELv2 means you can't offer Kreuzberg as a managed service -- fine for internal use, but it limits how you can build products on top of it. Its successor [[Xberg]] addresses this directly by switching to MIT/Apache 2.0 dual licensing, removing the managed-service restriction. Compare with [[Dolphin]] (document parsing) which is more narrowly focused but potentially more permissively licensed for production use.

The breadth-vs-depth tradeoff is real. Supporting 97+ formats means some will be better than others. For critical production use, you'd want to benchmark Kreuzberg against specialized tools for your specific format (e.g., [[Dolphin]] for scanned documents, dedicated PDF extractors for digital PDFs).

Still, for a "just extract text from whatever the user throws at me" use case, nothing else comes close to this coverage.

---
*Sources: [[summary/kreuzberg]]*
*Last updated: 2026-05-14*
