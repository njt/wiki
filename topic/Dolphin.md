# Dolphin

ByteDance's universal document parsing model. Handles any document type -- digital-born or photographed -- through a two-stage architecture that classifies the document type first, then applies the right parsing strategy. Accepted to ACL 2025.

---

## Key Themes

#ai #document-processing #ocr #vision-language-model

The two-stage architecture is the key design decision. Stage 1 classifies documents as digital or photographed and performs layout analysis with reading order prediction. Stage 2 applies different strategies depending on the result: holistic parsing for photographs (where OCR and layout are entangled), parallel element-wise parsing for digital documents (where text is already machine-readable and layout is structural).

This split makes sense. A photographed whiteboard and a digital PDF are fundamentally different parsing problems, and trying to handle them with one strategy means compromising on both. Dolphin v2 (3B parameters) scores 89.78 on OmniDocBench, a substantial jump from the 0.3B v1.5 model at 85.06.

21-element detection means it can identify and parse text blocks, tables, formulas, code, figures, captions, headers, footers, and more as distinct elements. The parallel processing with configurable batch sizes is important for production throughput.

## Critical Analysis

Compared to [[Kreuzberg]] (which wraps many extraction backends including VLMs), Dolphin is a single focused model. The tradeoff: Kreuzberg handles 97+ formats through multiple backends, Dolphin handles images and PDFs through one model. For pure document parsing quality, a focused model should win. For format coverage, the multi-backend approach wins.

The TensorRT-LLM and vLLM acceleration support signals production intent. This isn't just a research model -- ByteDance built it for deployment.

The single VLM backbone means the model is relatively simple to deploy compared to multi-model pipelines, but 3B parameters still requires meaningful compute. The 0.3B version trades quality for accessibility.

Another strong open-source release from ByteDance alongside [[Capybara]]. They're building a credible portfolio of production-grade AI models.

---
*Sources: [[summary/dolphin]]*
*Last updated: 2026-05-14*
