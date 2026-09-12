---
title: "graphify"
url: https://github.com/safishamsi/graphify/
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Graphify: Codebase to Knowledge Graph

Open-source Python tool transforming code, documentation, and media into queryable knowledge graphs. Three outputs: HTML visualization, markdown report, JSON data.

Supports 29 programming languages plus docs, PDFs, images, video/audio. Code processed locally via tree-sitter (nothing leaves your machine). Video/audio uses local faster-whisper. Only non-code content requires API credentials.

Graph analysis: Identifies "god nodes" (most-connected concepts), detects surprising cross-module connections, extracts design rationale from comments/docstrings, generates research questions, tags relationships with confidence levels (extracted, inferred, ambiguous).

Fully multimodal: code, PDFs, markdown, screenshots, diagrams, whiteboard photos, images in other languages. Uses Claude vision to extract concepts and relationships.

Team workflows: One person builds graph, commits to git, teammates' assistants access immediately. Automatic rebuilding via git hooks, conflict-free merge strategies.

47.6k stars, 5.2k forks. MIT licensed. 97 releases.
