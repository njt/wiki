---
title: "docmason"
url: https://github.com/jetxu-llm/docmason
date_fetched: 2026-05-14
section: "Random"
---

# DocMason: Local AI Knowledge Base for Office Documents

Repo-native agent application transforming office files into a locally-managed, traceable knowledge base. Every answer must be verifiable against original source documents.

Problem: Most document AI tools flatten complex files into unstructured text, losing presentation layouts, spreadsheet relationships, formatting-based semantics, cross-document connections.

Solution: Preserves original document structure, enforces strict source boundaries. "The repo holds the truth. The agent does the reasoning."

Runs entirely locally on macOS through Codex or Claude Code. Requires LibreOffice for high-fidelity Office parsing, Python 3.13.

Supported formats: PDF, PPTX, DOCX, XLSX, Markdown, plain text, email.

"Enforces strict data contracts and provenance boundaries." "Answers must be strictly traceable."

Workflow: Drop files into original_doc/, open in AI agent, prepare environment, build knowledge base, query with full source attribution.

Privacy: Zero transmission of document content, queries, or answers. All inference uses host agent's infrastructure.

Alpha status. Recommends GPT 5.4 or equivalent for reliable answers.
