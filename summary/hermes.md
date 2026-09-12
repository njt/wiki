---
title: "Hermes"
url: https://github.com/NousResearch/hermes-agent
date_fetched: 2026-05-14
section: "Personal Agents"
topics:
  - agent-memory-and-context
  - personal-agents
---

# Hermes Agent Overview

## Primary Purpose
Hermes Agent is a self-improving AI system developed by Nous Research that operates as an autonomous assistant with built-in learning capabilities. It's designed to function across multiple platforms and deployment environments.

## Core Features

**Learning Loop**: The system creates and refines skills from experience, maintains persistent memory across sessions, and searches past conversations for knowledge continuity.

**Multi-Platform Access**: Users can interact via CLI terminal interface, Telegram, Discord, Slack, WhatsApp, Signal, and Email through a unified gateway process.

**Model Flexibility**: Supports any LLM provider including Nous Portal, OpenRouter, NVIDIA NIM, OpenAI, Anthropic Claude, and custom endpoints with seamless switching.

**Deployment Options**: Runs on personal laptops, $5 VPS servers, GPU clusters, or serverless infrastructure (Daytona, Modal) that hibernates when idle.

**Terminal Interface**: Features a full TUI with multiline editing, command autocomplete, conversation history, streaming tool output, and interrupt capabilities.

## Key Capabilities

- **40+ integrated tools** for various tasks
- **Scheduled automation** via built-in cron scheduler
- **Parallel processing** through subagent delegation
- **Seven terminal backends** (local, Docker, SSH, Singularity, Modal, Daytona, Vercel)
- **Research-ready** batch trajectory generation and RL environment compatibility
- **MCP integration** for extending capabilities with external tools

## Technical Details

**Language Composition**: Primarily Python (87.9%), TypeScript (8.9%), with smaller portions of TeX, Shell, and Nix.

**Installation**: One-line installers for Linux, macOS, WSL2, Termux, and native Windows (early beta).

The repository contains 8,202 commits and maintains 149k stars on GitHub with active community engagement through Discord and a Skills Hub at agentskills.io.
