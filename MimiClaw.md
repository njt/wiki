# MimiClaw

An AI assistant running on a ~$5 ESP32-S3 microcontroller. Pure C, no Linux kernel, no Node.js, no cloud infrastructure. Connects to Anthropic or OpenAI APIs over WiFi, uses Telegram as its chat interface, and implements a ReAct agent loop with tool calling, persistent memory (SOUL.md, MEMORY.md), cron scheduling, and autonomous heartbeat tasks. 5.4k stars, which tells you something about the appeal of "AI on a chip."

---

## Key Themes

#embedded #esp32 #hardware #minimal #telegram

This is the logical extreme of the "how little infrastructure do you actually need?" question that runs through [[What I learned building an opinionated and minimal coding agent]] and [[PiClaw]]. Zechner stripped a coding agent to four tools; MimiClaw strips the runtime to a microcontroller. The answer to "can an AI agent run on $5 of hardware?" is yes, if you're willing to let the cloud do the inference and the chip just handles orchestration.

The architecture choices are forced by the hardware: dual-core processing (network I/O separate from AI computation), NVS flash for config persistence, plain-text files for memory. The SOUL.md / MEMORY.md / HEARTBEAT.md pattern for personality, memory, and autonomous tasks is charmingly simple -- and maps directly to the same concepts in [[Elements of Agentic Systems Design]] (Context, Memory, Autonomy).

## Critical Analysis

Strong: the 5.4k stars prove there's genuine interest in embedded AI agents. The 0.5W power consumption means you can run this indefinitely on USB. The runtime-switchable dual-provider support (Claude/GPT) without recompilation is impressive for pure C on a microcontroller.

Weak: the agent's capabilities are limited by ESP32 constraints -- no local inference, no file system to speak of, no browser. It's a chat bot with scheduling, which is useful but narrow. The Telegram dependency means another service you don't control. WiFi-only connectivity limits deployment scenarios.

The 5.4k stars suggest this taps into a different motivation than practical utility: the satisfaction of running AI on impossibly small hardware. That's fine -- exploration of constraints often produces insights that apply at larger scales.

---
*Sources: [[raw/mimiclaw]]*
*Last updated: 2026-05-14*
