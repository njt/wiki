---
title: "agentsview"
url: https://github.com/wesm/agentsview
date_fetched: 2026-05-14
section: "LLMs"
---

# AgentsView: Local-First Agent Session Intelligence

Fast local coding agent session viewer. "Browse, search, and track costs across all your AI coding agents. One binary, no accounts, everything local."

## Key Features

### Session Analytics & Search
- Full-text search across message content using FTS5
- Token usage and cost dashboards with per-session and per-model breakdowns
- Activity heatmaps and velocity metrics
- HTML export and GitHub Gist publishing

### Cost Tracking
Automatic pricing via LiteLLM rates with offline fallback. Prompt-caching-aware cost calculations. Per-model breakdowns. JSON output for scripting.

### Analytics Dashboard
Session archetypes, context distribution metrics, tool/model/agent mix analysis, temporal hourly breakdowns with optional git-derived outcome metrics.

## Supported Agents

Auto-discovers sessions from 24+ agents: Claude Code, Codex, Cursor, OpenHands, VSCode Copilot, Gemini CLI, and more.

## Architecture

- Backend: Go 1.26+ (71.3%)
- Frontend: Svelte 5 with Vite and TypeScript
- Desktop: Tauri wrapper
- Database: SQLite with FTS5; optional PostgreSQL sync
- Live updates via Server-Sent Events
- Keyboard-first navigation (j/k, Cmd+K)
- 100x faster queries than similar tools (indexed SQLite)

## Privacy

No telemetry, no analytics, no user accounts. Only outbound request is optional update check (disable with --no-update-check).
