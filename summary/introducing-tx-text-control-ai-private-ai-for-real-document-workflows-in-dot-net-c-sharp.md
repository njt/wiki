---
url: https://www.textcontrol.com/blog/2026/09/09/introducing-tx-text-control-ai-private-ai-for-real-document-workflows-in-dot-net-c-sharp/
title: "Introducing TX Text Control AI: Private AI for Real Document Workflows in .NET C#"
author: Text Control (vendor blog, no byline)
date_fetched: 2026-09-13
date_published: 2026-09-09
topics:
  - mcp-and-tool-protocols
  - local-and-open-source-inference
---

Text Control announces the TX Text Control AI Preview (0.1.0-beta.1): six NuGet packages and ten samples that wire local generative AI, private knowledge retrieval, and the TX Text Control document engine together in .NET. The architecture rests on a three-way separation: the model handles language, an integration layer manages workflow, and the document engine performs deterministic document operations. The model never manipulates document binaries — it proposes structured MCP tool calls, and the MCP document server (the component that "knows how to work with document formats, native paragraph styles, tables, fields, and exports") executes them.

The packages split cleanly: `TXTextControl.AI` is the developer-facing `Microsoft.Extensions.AI.IChatClient`-based API; `.AI.LlamaServer` manages a native llama.cpp server process, hardware reporting, and an embedding generator; `.AI.Mcp` is the client side; `.AI.AspNetCore` ships reusable HTTP endpoints and browser libraries; `.AI.Knowledge` provides private collections, versioned sources, SQLite full-text search, and optional semantic/hybrid retrieval; `.AI.McpServer` hosts the document engine separately. Deployment can spread across three hosts — web UI, AI service (GPU inference + Knowledge), and MCP document server — with authenticated proxies between them and credentials kept server-side.

The private RAG story is deliberately modest: uploading documents does not train the model, references are read-only by default, keyword search is immediate while semantic search uses a separate Qwen3-Embedding-0.6B GGUF, and hybrid retrieval combines both rankings. Long documents travel through the document workflow rather than as Base64 in the conversation, with hierarchical chunked analysis of extracted text instead. The post is unusually candid for a launch announcement: local does not mean secure or compliant, the sample loads Google Fonts by default, a filename is not a benchmark, retrieval is not proof of completeness, and the sample identity plus single-host SQLite knowledge storage are "starting points, not complete enterprise, multi-tenant deployments."
