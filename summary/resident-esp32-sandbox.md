---
url: https://resident.inanimate.tech/
title: Resident — Sandbox Runtime for ESP32 Devices
author: INANIMATE (inanimate.tech)
date_fetched: 2026-05-22
date_published: 2026-05-20
topics:
  - security-and-sandboxing
---

# Resident — Sandbox Runtime for ESP32 Devices

Published by INANIMATE (inanimate.tech), with a cross-reference to an interconnected.org article dated May 20, 2026.

## Overview

Resident is a code sandbox with hot reload for ESP32 devices. It supports both esp-idf and Arduino frameworks and is released under an MIT Open Source License. The tagline positions it as an alpha-stage project.

## Core Concept

The project's central argument is that on-device sandboxes are necessary for AI agents operating in the physical world: "Devices need sandboxes for agents. AI is coming into the real world."

The architecture connects several layers:

- **Device** — ESP32 with connectivity and custom hardware
- **Driver** — C++ extensions exposing hardware to Lua
- **Sandbox** — Lua runtime with hot-reloading and isolation
- **App** — Lua apps loadable over the network and shareable by users
- **Events** — from hardware and network via JSON over WebSockets or MQTT

## Key Features

- Lua sandbox with hardware IO — drivers expose approved peripherals, the sandbox consumes their events
- WebSocket-based connectivity for pushing and iterating apps
- Hot-reloading Lua apps over the network
- Integration with both Arduino and esp-idf frameworks
- Built on Courier for "batteries-included connectivity including Wi-Fi config and JSON messaging"
- Includes a browser-based M5StickS3 simulator for testing without hardware

## Workflow

Three-step build process: (1) bring up an ESP32 device normally; (2) add the sandbox with custom drivers; (3) create and push apps using built-in Claude skills. The page provides an agent-facing CLI flow involving a plugin marketplace.

## Design Philosophy

1. **Sandboxes as a new primitive** — The team uses Resident in all their prototyping, noting that "users need expressive control of their environments, and agents need a place to run code to make generative UI."

2. **Composability** — Built to integrate "with existing firmware, frameworks, and back-end services." ESP32 was chosen because "it's used by makers, hardware startups, and in mass production."

3. **Built for agents** — The project provides agent-facing docs and skills for the full app creation lifecycle. The page notes that "AI agents are how we code now, and also the emerging interface for end users," with the goal of making it "easy for agents to hermit crab behavior into the world."

## Example Apps

Sample Lua apps include `lil-guy.lua`, `auto-tetris.lua`, and `pomodoro.lua` — demonstrated on an M5StickS3 simulator running in the browser.

## License

MIT Open Source License.
