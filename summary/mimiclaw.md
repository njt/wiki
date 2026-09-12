---
title: "mimiclaw"
url: https://github.com/memovai/mimiclaw
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - personal-agents
  - agent-architecture
---

# MimiClaw: AI Assistant on ESP32

Runs an AI assistant on a ~$5 ESP32-S3 microcontroller. Pure C, no Linux kernel, no Node.js, no cloud dependencies.

## Hardware

- ESP32-S3 with 16MB flash and 8MB PSRAM
- USB-powered (0.5W)
- WiFi-connected via Telegram messaging interface
- WebSocket gateway on port 18789

## AI Capabilities

- Dual-provider: Anthropic (Claude) and OpenAI (GPT)
- ReAct agent loop with tool calling for both providers
- Switchable at runtime without recompilation
- Web search via Tavily or Brave Search API

## Persistent Storage & Memory

Plain-text files:
- SOUL.md: personality/behavior definition
- MEMORY.md: long-term retention across reboots
- HEARTBEAT.md: autonomous task checking (~30-minute intervals)
- cron.json: scheduled job persistence

## Autonomous Features

Built-in cron scheduler for recurring/one-shot tasks. Heartbeat service prompts independent action. Session-based chat history per user.

## Architecture

Dual-core processing: network I/O independent from AI computation. Two-layer config: build-time defaults in mimi_secrets.h with runtime CLI overrides persisted to NVS flash.

Tools: web_search, get_current_time, cron_add/list/remove.

## Stats

C (96.6%), ESP-IDF v5.5+, 214 commits, 5.4k stars, 790 forks.
