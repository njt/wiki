# Gemma Gem

A Chrome extension that runs Google's Gemma 4 language model locally in the browser via WebGPU. No cloud, no API keys, no data leaving your machine. It can read pages, click buttons, fill forms, execute JavaScript, and answer questions -- all on-device. Two model sizes: E2B (~500MB, 4GB VRAM) and E4B (~1.5GB, 6GB VRAM).

---

## Key Quotes

> No standout quotes -- it's a tool README.

## Key Themes

#on-device-ai #webgpu #browser-extension #privacy #local-llm

The on-device angle is the differentiator. While [[Browser Use]] routes everything through cloud APIs with stealth infrastructure, Gemma Gem keeps everything local. Zero data transmission means zero privacy concerns -- but also zero access to larger, more capable models. It's the privacy-first alternative.

The WebGPU dependency limits reach (Chrome 113+ with GPU support), but the trajectory is clear: browser-based AI inference is becoming practical for modest-sized models. As models shrink and hardware improves, this approach scales.

## Critical Analysis

The practical question is whether E2B/E4B models are capable enough for useful browser automation. Gemma 4's smaller variants are competent for simple tasks but struggle with complex multi-step workflows. For reading a page and answering questions, probably fine. For the kind of multi-step form-filling and data extraction that [[Browser Use]] handles, probably not yet.

The architecture (offscreen document for inference, service worker for routing, content script for DOM) is clean and well-separated. The HuggingFace Transformers.js dependency is the right choice for browser-based inference. Worth watching as models improve -- the on-device browser agent is a compelling end state even if it's not fully capable today.

---
*Sources: [[summary/gemma-gem]]*
*Last updated: 2026-05-14*
