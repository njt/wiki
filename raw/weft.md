---
title: "weft"
url: https://github.com/jonesphillip/weft
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Weft: AI Agent Task Management

Personal task board where AI agents autonomously work on assigned tasks. Users create tasks, assign them to agents, and agents execute work like reading emails, drafting responses, updating spreadsheets, creating pull requests, and writing code.

## Key Features
- Parallel agent execution: Run multiple agents simultaneously
- Scheduled tasks: Daily, weekly, or custom cron expressions
- Approval-based safety: All state-mutating actions require human approval before execution
- Built-in integrations: Gmail, Google Docs, Google Sheets, GitHub, Cloudflare Sandbox, remote MCP servers

## Architecture
Runs entirely on Cloudflare's Developer Platform:
- Workers: HTTP routing, authentication, React frontend
- Durable Objects: Persistent state management and real-time WebSocket updates
- Workflows: Durable agent loop with automatic checkpointing and retry logic

## Notable Quote
"Task management, but AI agents do your tasks. Self-host on Cloudflare."

Self-hosted, data stays private. TypeScript (81.5%), Apache 2.0 license.
