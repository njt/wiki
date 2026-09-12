# QMD

A mini CLI search engine for your docs, knowledge bases, meeting notes, whatever. All local, tracking current SOTA approaches. Hybrid search combining BM25, vector embeddings, and LLM re-ranking.

---

## Key Themes

#tool #search #local-first #knowledge-management #mcp

The architecture is more sophisticated than "local search engine" suggests. The pipeline runs query expansion (LLM generates variations of your search), parallel retrieval against both keyword (BM25) and vector indexes, Reciprocal Rank Fusion to merge results, and finally LLM re-ranking with confidence scores. This is the same architecture used by production RAG systems at scale, but running entirely on your laptop.

Three GGUF models auto-download on first use (~2GB total): EmbeddingGemma for vectors, Qwen3-Reranker for relevance scoring, and a custom query-expansion model. Smart chunking preserves semantic units -- headings, code blocks, and sections stay together rather than cutting at arbitrary token boundaries.

The MCP server integration means Claude (and other LLM clients) can use QMD as a tool for searching your local documents. The multiple output formats (JSON, CSV, Markdown, XML) make it agent-friendly.

## Critical Analysis

Nat's own caveat on this one: "All local is a great idea, but I end up sending everything upstream to a big model anyway so privacy is less important to me than performance in this case." That's an honest assessment. Local search is a privacy win but a quality tradeoff -- cloud-based retrieval with larger models will generally produce better results.

The 2GB model download is a one-time cost but it's not nothing. And the Node.js >= 22 requirement limits deployment on older systems.

Where QMD shines is the MCP integration. If you're already using Claude Code and want it to search your local notes without uploading them to an API, QMD is the most turnkey solution. The "context" layers that add metadata for improved ranking are a nice feature for people who organize their documents carefully.

By Tobi Lutke (Shopify CEO), which explains the polish. This isn't a weekend project.

---
*Sources: [[summary/qmd]]*
*Last updated: 2026-05-14*
