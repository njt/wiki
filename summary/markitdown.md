---
title: "markitdown"
url: https://github.com/microsoft/markitdown
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# MarkItDown (Microsoft)

Lightweight Python utility for converting various file formats into Markdown. Designed for LLMs and text analysis pipelines, prioritizing document structure preservation over human-friendly presentation.

Supported formats: PDF, PowerPoint, Word, Excel, images (EXIF + OCR), audio (speech transcription), HTML, CSV, JSON, XML, ZIP, YouTube URLs, EPub.

Key arguments for Markdown: "Markdown is extremely close to plain text, with minimal markup or formatting, but still provides a way to represent important document structure." Mainstream LLMs natively understand Markdown and often use it unprompted. Markdown conventions are "highly token-efficient."

Features: Plugin support, Azure Document Intelligence integration, LLM integration for image descriptions (OpenAI-compatible), Docker support, CLI and Python API.

Security: Performs I/O with current process privileges. Sanitize inputs in untrusted environments.

Python 3.10+. Optional dependencies for specific formats.
