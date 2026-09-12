---
title: "AI Brain for Flipper"
url: https://github.com/elder-plinius/V3SP3R
date_fetched: 2026-05-14
section: "Random"
topics:
  - security-and-sandboxing
---

# V3SP3R: AI-Powered Flipper Zero Control

Android app that transforms Flipper Zero into an AI-controlled command center. Natural language interaction via voice, text, and images through OpenRouter's language models.

Eliminates menu navigation and manual signal crafting. Issue commands like "Show me my SubGHz captures" or "Generate a BadUSB script."

Hardware integration: SubGHz RF transmission/analysis, infrared commands, NFC/RFID/iButton emulation, BadUSB HID attacks, GPIO/LED/vibration control.

Safety architecture:
- Risk classification for all AI actions
- Low-risk: auto-execute
- Medium-risk: show diffs for review
- High-risk: require explicit confirmation
- Protected system paths require manual unlock

Additional: Alchemy Lab (custom RF signal synthesis), Payload Lab (AI-generated badUSB/SubGHz/IR artifacts), FapHub app browser, GitHub resource discovery, device diagnostics, complete audit logging.

Tech: Kotlin (39.8%) + Java (48.2%), Jetpack Compose, Hilt DI, Room Database with encrypted DataStore.

Recommended models: Hermes 4 (tool-use), Claude Opus 4.6 (reasoning), Claude Sonnet 4 (balanced default).

Smart glasses integration via optional Mentra Node.js bridge.

AGPL-3.0 licensed. 1k stars, 184 forks.
