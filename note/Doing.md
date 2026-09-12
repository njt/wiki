# Doing

Fast local voice transcription for Mac. $49 one-time purchase, no subscription, no cloud processing. Uses NVIDIA's Parakeet model on-device to transcribe 60 seconds of audio in approximately 400 milliseconds -- "150x realtime." Your voice never leaves your machine.

---

## Key Quotes

> "Stop editing, stop polishing, just talk. LLMs know what you mean, even when your words aren't perfect."

> "Own your tools, stop renting."

> "Other voice tools charge $8-15 a month, forever. Doing is $49 once."

## Key Themes

#tool #voice #transcription #privacy #local-first

The "own your tools" philosophy is the real pitch. In a world of SaaS subscriptions, a one-time purchase for a productivity tool that runs entirely on your hardware is refreshing. The privacy story writes itself: no audio uploaded, no account required, no cloud transcription.

The performance claim of 150x realtime is striking. If true (and local Whisper-class models are genuinely this fast on Apple Silicon), the latency argument for cloud transcription evaporates. The only remaining argument for cloud is accuracy on unusual accents or specialized vocabulary, and that gap is closing.

The "LLMs know what you mean" line is the key insight for the voice-to-AI-agent pipeline. You don't need perfect transcription when the downstream consumer is an LLM that can handle messy input. This changes the quality bar for transcription tools.

## Critical Analysis

Mac-only limits the audience. The $49 price point is smart -- it's low enough to be impulse-buy territory but high enough to fund development. Compare with [[Handy]], which is free and open-source but less polished.

The YOLO Mode (auto-press return after pasting) is a telling feature -- it's designed for the "talk to your terminal" workflow where you dictate commands or prompts directly. This is the voice-driven agent interaction pattern that's emerging.

Missing: multi-language support details, accuracy benchmarks against competitors, and whether it handles code dictation (variable names, syntax) gracefully.

---
*Sources: [[summary/doing]]*
*Last updated: 2026-05-14*
