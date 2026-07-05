# docmason

A repo-native agent application that transforms office documents into a locally-managed, traceable knowledge base. The core principle: every answer must be verifiable against original source documents. "The repo holds the truth. The agent does the reasoning."

---

## Key Quotes

> "The repo holds the truth. The agent does the reasoning."

> "Enforces strict data contracts and provenance boundaries."

## Key Themes

#tool #document-analysis #provenance #citations #local-first

The problem docmason addresses is real: most document AI tools flatten complex files into unstructured text, losing the structural information that makes answers verifiable. Presentation layouts, spreadsheet relationships, formatting-based semantics, and cross-document connections all disappear. You get plausible answers with no way to check them.

docmason's approach: preserve original document structure, enforce strict source boundaries, and make every AI answer traceable to specific source material. It uses LibreOffice for high-fidelity parsing (important -- Office format conversion is notoriously lossy) and runs entirely locally on macOS.

The "one of many tools doing citations and source weighting" observation in the annotation captures a trend. As LLMs become more capable at analysis, the bottleneck shifts to trustworthiness. Provenance and citation become the differentiator.

## Critical Analysis

Alpha status and a recommendation for "GPT 5.4 or equivalent" for reliable answers suggests this is early. The concept is strong but the execution may not be there yet.

The local-only approach (zero transmission of content) is the right call for corporate documents but limits the model quality you can use. Unless you're running a capable local model, you're sending content through whatever API your agent uses.

Compare with [[markitdown]], which converts documents to flat Markdown (good for feeding into LLMs, bad for preserving structure), and [[graphify]], which builds knowledge graphs from code (similar concept, different domain). docmason occupies the middle ground: structured knowledge extraction from office documents.

See also [[lat.md]] for another approach to validated, traceable knowledge bases.

---
*Sources: [[summary/docmason]]*
*Last updated: 2026-05-14*
