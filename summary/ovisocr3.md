---
url: https://huggingface.co/ATH-MaaS/OvisOCR3
title: "OvisOCR3"
author: ATH-MaaS
date_fetched: 2026-10-24
date_published: 2026-10-24
topics:
  - ai-research-and-models
  - local-and-open-source-inference
---

OvisOCR3 is a unified 0.8B multimodal model for OCR and document understanding from ATH-MaaS, built on Qwen3.5-0.8B and trained in four stages (continued pretraining, mid-training, on-policy distillation, reinforcement learning). Five document-centric capabilities — document parsing, general OCR, key information extraction, document figure captioning, and cross-page table reconstruction — are selected through task-specific prompt files rather than separate deployments.

The model card reports state-of-the-art results among models of comparable size across all five domains: document-parsing benchmarks (OmniDocBench1.6 96.74, MDPBench 88.80), general OCR (OCRBench 88.8, OCRBench v2 EN 65.9 — well ahead of larger Qwen3.5-2B/4B), KIE, figure captioning (DocFigCapBench localization F1 90.97), and a striking lead on cross-page table reconstruction (CrossTabBench relation F1 97.07 vs. 94.47 for Qwen3.8-Max).

The bulk of the page is engineering documentation rather than marketing: three parsing modes (Markdown, structured JSON with bounding boxes, layout-only), ten general-OCR tasks, three KIE modes, a two-stage cross-page table pipeline with recursive pairwise merging for longer tables, and explicit postprocessing code (repeated-tail cleanup, image-crop extraction from bbox references). Coordinate systems are normalized to [0,1000) across all tasks. vLLM serving examples are provided; the license is Apache 2.0.

The limitations section is unusually honest: outputs can be truncated or malformed, schema adherence is "prompted, not enforced by constrained decoding," HTML and Mermaid output must be treated as untrusted and sanitized before rendering, and the usage examples explicitly do not define a benchmark protocol.
