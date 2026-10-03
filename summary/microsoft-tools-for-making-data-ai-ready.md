---
url: https://www.sqlservercentral.com/blogs/microsoft-tools-for-making-data-ai-ready
title: "Microsoft Tools for Making Data AI-Ready (Part 2)"
author: James Serra
date_fetched: 2026-10-03
date_published: unknown (2026, part 2 of a three-part series)
topics:
  - databases-and-data
  - agent-memory-and-context
---

James Serra's second article in a three-part series maps the Microsoft toolchain for making data usable by AI systems, organised by the problem each tool solves rather than as a shopping list. Physical preparation comes first: Fabric Dataflow Gen2 (Power Query), notebooks, Data Factory pipelines, and SQL views clean and shape data into curated lakehouse/warehouse tables — fix the actual data rather than instructing an agent to work around known problems.

Reusable business meaning is next. A Power BI semantic model supplies friendly names, relationships, and explicit measures (Customers exposed even though the table is `People`), while Microsoft Purview provides governance metadata — cataloging, ownership, lineage — though Serra warns not to assume a glossary definition automatically reaches an agent's prompt. Fabric IQ Ontology (preview) adds a business-meaning layer binding concepts like Customer and Order to physical data, so questions like "which customers bought Outdoor products" have an explicit business path instead of re-explanations in every prompt.

For AI-specific context, Power BI's Prep data for AI adds AI data schemas, AI instructions, and Verified Answers to a semantic model; Fabric Data Agent configuration supplies agent instructions, source descriptions, and example queries, with support varying by source type. Third-party BI Pixie scores semantic-model AI readiness, making readiness something measurable rather than assumed. For documents, Azure AI Search integrated vectorization and Foundry IQ knowledge bases handle retrieval preparation, with full-text, hybrid, and semantic ranking as complementary choices. The closing recommendation: work backward from a specific question and failure — cleaning, modeling, definitions, or retrieval — rather than adopting every tool.
