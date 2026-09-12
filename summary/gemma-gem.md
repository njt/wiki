---
title: "Gemma Gem"
url: https://github.com/kessler/gemma-gem
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - personal-agents
---

# Gemma Gem: On-Device AI Assistant

Chrome extension running Google's Gemma 4 locally in the browser via WebGPU. No cloud services, no API keys.

## Capabilities
- Reads and analyzes webpage content
- Clicks buttons, interacts with page elements
- Fills forms, executes JavaScript
- Answers questions about websites
- All processing on-device, no data transmission

## Models
- Gemma 4 E2B (~500MB storage)
- Gemma 4 E4B (~1.5GB storage)

## Architecture
1. **Offscreen Document** -- hosts model via HuggingFace Transformers + WebGPU, manages agent loop and token streaming
2. **Service Worker** -- routes messages, handles screenshots and JS execution
3. **Content Script** -- chat interface and DOM tools

## Tools
Read page content (CSS selectors), capture screenshots, click elements, type text, scroll, execute JavaScript.

## Requirements
- E2B: 4GB GPU VRAM, 6-8GB system RAM
- E4B: 6GB GPU VRAM, 8-16GB system RAM
- Chrome 113+ or Edge 113+ with WebGPU

Built with WXT (Vite), HuggingFace Transformers.js, marked.
