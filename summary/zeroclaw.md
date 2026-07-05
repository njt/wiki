---
title: "Zeroclaw"
url: https://github.com/zeroclaw-labs/zeroclaw
date_fetched: 2026-05-14
section: "LLMs"
---

# ZeroClaw: Personal AI Assistant Infrastructure

Rust-based agent runtime. Single binary. 31.3k GitHub stars, dual MIT/Apache 2.0 license.

"You own the agent. You own the data. You own the machine it runs on."

## Core Capabilities

### Multi-Channel Communication
Handles messages across 30+ channels: Discord, Telegram, Matrix, email, webhooks, CLI. All routed through a unified agent loop.

### Provider Flexibility
Multiple LLM providers (Anthropic, OpenAI, Ollama, ~20 others) with configurable fallback chains and routing for provider outages.

### Security Architecture
Supervised autonomy: medium-risk operations require approval, high-risk blocked. Workspace boundaries, command policies, OS-level sandboxes (Landlock, Bubblewrap, Seatbelt, Docker), cryptographic tool receipts documenting every action.

### Hardware Integration
GPIO, I2C, SPI, USB support for Raspberry Pi, STM32, Arduino, ESP32 via a Peripheral trait abstraction.

### Additional Features
- HTTP/WebSocket gateway with web dashboard
- Standard Operating Procedures (SOP) engine for event-triggered automation
- IDE integration via Agent Client Protocol (JSON-RPC 2.0)

## Architecture

Everything is interface-based (Rust traits), so you can drop in replacements for any component by honoring the same interface. Channels/gateway/ACP feed into a core loop handling agents, security policy, and SOPs, connecting down to providers, tools (shell, browser, HTTP, hardware), and memory.

## Installation

Automated installer supporting prebuilt binaries and source builds, with feature customization and platform-specific guidance for Linux, macOS, Windows, Docker.
