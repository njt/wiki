---
title: "Dolphin"
url: https://github.com/bytedance/Dolphin
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Dolphin: Universal Document Parsing Model
By ByteDance (accepted to ACL 2025)

Enhanced universal document parsing model (v2). Handles any document type -- digital-born or photographed -- through a document-type-aware two-stage architecture with scalable anchor prompting.

## Architecture
**Stage 1**: Document classification (digital vs. photographed) alongside layout analysis with reading order prediction.
**Stage 2**: Hybrid parsing -- holistic parsing for photographed documents, parallel element-wise parsing for digital documents.

## Key Capabilities
- Page-level parsing: entire pages to structured JSON and Markdown
- Element-level parsing: individual components (text, tables, formulas, code)
- Layout analysis: document structure with natural reading order
- Multi-format: PNG images and PDF documents
- 21-element detection
- Parallel processing with configurable batch sizes

## Performance (OmniDocBench)
Dolphin-v2 (3B parameters): overall 89.78
Dolphin-1.5 (0.3B): overall 85.06

## Technical Details
- Single Vision Language Model (VLM) backbone
- Heterogeneous anchor prompting for varied element types
- HuggingFace Transformers integration
- TensorRT-LLM and vLLM acceleration
- Multi-page PDF parsing
