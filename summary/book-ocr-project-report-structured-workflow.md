---
url: https://parc.yolo.scapegoat.dev/note/projects/2026/05/26/article-book-ocr-project-report-structured-workflow-runtime-and-manual-pdf-repair
title: "Book OCR Project Report — Structured Workflow Runtime and Manual PDF Repair"
author: Manuel (wesen/go-go-golems)
date_fetched: 2026-05-31
date_published: 2026-05-26
date_modified: 2026-05-31
tags:
  - article
  - ocr
  - workflow
  - geppetto
  - pinocchio
  - structured-output
  - pdf
  - book-processing
aliases:
  - Book OCR Project Report
  - Report 794 OCR Project
  - Structured Book OCR Full Project Report
  - Workflow Backed Book OCR
repo: /home/manuel/workspaces/2026-05-20/book-ocr/2026-05-20--book-ocr
topics:
  - software-engineering-craft
---

Full article text fetched via surf browser automation (JS-rendered Obsidian Publish page). WebFetch returned only site title "Retro Obsidian Publish" with no body content.

The article is a detailed project report on building a structured OCR pipeline for a 202-page scanned technical book ("Report 794, Presentation Based User Interfaces"). The project evolved from a freeform Markdown OCR experiment into a full workflow-backed production system with structured JSON output, deterministic Markdown rendering, figure extraction, PDF generation, validation gates, and targeted page repair.

Key architecture: Go-based workflow runtime (`scraper/pkg/workflow`) providing durable execution with steps, queues, leases, retries, artifacts, and projections. OCR application (`book-ocr`) uses Geppetto/Pinocchio for vision model calls with target-page-only image input and structured JSON output contracts. Renderer converts structured OCR blocks into deterministic Markdown. Figure extraction handles crops and embedding. Validation checks page count, duplicate captions, short pages, and code-fence compliance.

Central insight: model output should stop at a structured boundary (JSON), not freeform Markdown. This separation let Go own deterministic rendering and validation, making targeted page repairs possible without rerunning the entire book.

Seven engineering rules derived: target-page-only OCR, structured model boundaries, preserve raw evidence before parsing, separate workflow state from domain projection state, render the PDF from workflow state, support targeted repair, encode every repeated manual finding as a validation check.