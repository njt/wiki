---
title: "piclaw"
url: https://github.com/rcarmo/piclaw
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - personal-agents
  - agent-architecture
---

# PiClaw: Self-Hosted AI Workspace

Packages the Pi Coding Agent into a containerized environment with persistent state, multi-provider LLM support, and an integrated web UI. MIT licensed, primarily TypeScript.

## Core Features

### Unified Environment
Single web application combining chat, code editor, terminal, file viewers, and automation tools. SQLite-backed persistent storage for messages, media, tasks, and token usage.

### Agent Capabilities
Steering, queued follow-ups, side prompts, autoresearch loops, scheduled tasks, and visual artifact generation. Built-in tools for code editing, Office/PDF/CSV/image/video viewing, browser automation, and image processing. MCP support.

### Authentication & Access Control
Optional passkeys and TOTP for web UI security. WhatsApp integration option. Session-scoped SSH.

## Technical Architecture

- TypeScript (95.5%)
- Bun runtime
- Docker containerization with docker-compose
- Ghostty-based web terminal
- Azure VM deployment docs provided

## Deployment

Docker container (primary), global Bun install, experimental Electrobun desktop wrapper.

Quick-start uses two mount points: /config for agent home, /workspace for projects and state. Critical: "Never delete /workspace/.piclaw/store/messages.db" -- contains essential chat history and task data.
