---
title: "World Monitor"
url: https://github.com/koala73/worldmonitor
date_fetched: 2026-05-14
section: "Random"
---

# World Monitor: Real-Time Global Intelligence Dashboard

AI-powered news aggregation, geopolitical monitoring, and infrastructure tracking in a unified situational awareness interface.

## Key Features
- 500+ curated news feeds across 15 categories, AI-synthesized into briefs
- 65+ external data sources across geopolitics, finance, energy, climate, aviation, cyber intelligence
- Cross-stream correlation linking military, economic, disaster, and escalation signals
- Dual mapping engine: 3D globe (globe.gl) and flat WebGL map (deck.gl) with 45 geospatial layers
- Country Intelligence Index with composite risk scoring across 12 signal categories
- Finance radar covering 92 stock exchanges, commodities, and cryptocurrency
- 21 languages with native-language feeds and RTL support
- Native desktop app via Tauri 2 (macOS, Windows, Linux)
- Five site variants from single codebase: world, tech, finance, commodity, happy
- Local AI via Ollama (no API keys required)

## Tech Stack
- Frontend: Vanilla TypeScript, Vite, Three.js, MapLibre GL
- Backend/Desktop: Tauri 2 (Rust), Node.js sidecar
- AI/ML: Ollama, Groq, OpenRouter, Transformers.js
- Infrastructure: Vercel Edge Functions, Railway relay, Redis caching, PWA

54.1k stars. AGPL-3.0 for non-commercial use.
