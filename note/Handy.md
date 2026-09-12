# Handy

Free, open-source, cross-platform speech-to-text. Press a keyboard shortcut, speak, release -- it transcribes locally and pastes into whatever text field you're typing in. One tool, one job. Built with Tauri (Rust + React/TypeScript), supports macOS, Windows, and Linux.

---

## Key Quotes

> "Your voice stays on your computer. Get transcriptions without sending audio to the cloud."

> "One tool, one job. Transcribe what you say and put it into a text box."

> "Handy isn't trying to be the best speech-to-text app -- it's trying to be the most forkable one."

## Key Themes

#tool #voice #transcription #privacy #open-source #local-first

The "most forkable" positioning is the distinguishing feature. Where [[Doing]] is a polished commercial product, Handy is a community-extensible platform. It uses Whisper models (small through large with GPU acceleration) or Parakeet V3 (CPU-optimized with automatic language detection), giving users flexibility on the speed-accuracy tradeoff.

The four pillars -- Free, Open Source, Private, Simple -- are a statement about what accessibility tools should be. Voice transcription is an accessibility feature before it's a productivity tool, and paywalling it feels wrong. Handy takes that position explicitly.

Cross-platform support (macOS, Windows, Linux) via Tauri is a significant advantage over Mac-only alternatives. The Silero Voice Activity Detection filters silence automatically, which is important for push-to-talk workflows.

## Critical Analysis

21.6k stars suggests this hit a nerve. The Tauri architecture (Rust backend, React frontend) is modern and keeps the binary small. The real test is accuracy -- Whisper models vary dramatically by size, and the small model that runs fast on CPU may not be accurate enough for serious use. The turbo/large models with GPU acceleration close this gap but limit the "runs on anything" promise.

The push-to-talk default is the right choice for agent-interaction workflows. Continuous transcription modes create noisy input; deliberate dictation produces cleaner text.

See also [[Doing]] for the commercial alternative with different tradeoffs, and [[surf-cli]] for another tool that prioritizes the "just works" local experience.

---
*Sources: [[summary/handy]]*
*Last updated: 2026-05-14*
