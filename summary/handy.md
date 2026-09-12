---
title: "Handy"
url: https://github.com/cjpais/Handy
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
  - local-and-open-source-inference
---

# Handy: Offline Speech-to-Text

Free, open source, and extensible speech-to-text application that works completely offline.

## Core Philosophy
1. **Free**: Accessibility tools shouldn't be paywalled
2. **Open Source**: Community-driven extensibility
3. **Private**: Audio remains local, no cloud transmission
4. **Simple**: Single-purpose transcription and text insertion

"Handy isn't trying to be the best speech-to-text app -- it's trying to be the most forkable one."

## How It Works
1. Press a configurable keyboard shortcut (or push-to-talk)
2. Speak while the shortcut remains active
3. Release to trigger local processing
4. Transcribed text pastes directly into active application

Uses Voice Activity Detection (Silero) to filter silence. Transcription via Whisper models (small/medium/turbo/large with GPU acceleration) or Parakeet V3 (CPU-optimized with automatic language detection).

## Platforms
- macOS (Intel and Apple Silicon)
- Windows (x64)
- Linux (x64)

Built with Tauri (Rust backend, React/TypeScript frontend). 21.6k stars.
