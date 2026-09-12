---
title: "Dorothy"
url: https://dorothyai.app/
repo: https://github.com/Charlie85270/Dorothy
author: Charlie85270
date_fetched: 2026-05-14
date_published: 2025
topics:
  - agent-orchestration
---

# Dorothy: AI Agent Orchestration Platform

Free, open-source (MIT) desktop app for orchestrating multiple AI coding agents simultaneously. Built by Charlie85270. Tagline: "Your AI Agents, Perfectly Managed."

## Architecture (Three Layers)

### 1. Electron Desktop Shell
- **Renderer**: React 19 / Next.js 16 (App Router) — agent dashboards, Kanban, automations, scheduled tasks, usage stats, skills, plugins, settings
- **Main Process**: Agent Manager (node-pty-based), PTY Manager, background services (Telegram bot, Slack bot, Kanban automation, MCP server launcher, API server)
- **API Routes**: Next.js backend; renderer↔main IPC communication

### 2. Five MCP Servers (stdio-based)
- **mcp-orchestrator** (26+ tools): Agent lifecycle, messaging, scheduling, automation management, JIRA updates
- **mcp-telegram** (4 tools): Messaging with photo/video/document support
- **mcp-kanban** (8 tools): Programmatic CRUD for Kanban tasks
- **mcp-vault** (10 tools): Persistent document storage, SQLite FTS5 full-text search
- **mcp-socialdata** (5 tools): Twitter/X search, user profiles, engagement data

### 3. Agent Execution Layer
- Isolated `claude` CLI processes via `node-pty`
- Real-time output streaming per agent
- Status detected by parsing output patterns
- Autonomous execution supported (`--dangerously-skip-permissions`)
- Secondary project paths via `--add-dir` and git worktree support

## Core Features

### Parallel Agent Management
- Unlimited concurrent agents across projects
- Per-agent: isolated PTY, skill assignments, model selection (sonnet/opus/haiku)
- Lifecycle: idle → running → completed / error / waiting
- Persistent state across restarts (`~/.dorothy/agents.json`)

### Super Agent (Orchestrator)
Meta-agent that programmatically creates, starts, stops other agents via MCP tools. Delegates by skills/capabilities, monitors progress, spawns temporary agents for one-off tasks. Remote-controllable via Telegram and Slack.

### Automations
CI/CD-like AI workflows polling external sources on cron schedules:

| Source | Status | Method |
|--------|--------|--------|
| GitHub | Active | `gh` CLI (PRs, issues, releases) |
| JIRA | Active | REST API v3 |
| Pipedrive | Planned | — |
| Twitter | Planned | — |
| RSS | Planned | — |
| Custom | Planned | Webhooks |

Template variables (`{{title}}`, `{{url}}`, `{{body}}`, `{{labels}}`, etc.) inject item data into prompts. Content-hash deduplication prevents re-processing.

### Kanban Task Management
Columns: Backlog → Planned → Ongoing → Done. Priority levels, 0-100% progress, manual/auto agent assignment, skill requirements. The `kanban-automation` service continuously matches tasks to agents by skill — creating agents if none match. JIRA automations auto-create Kanban backlog tasks, forming a closed loop from external issues to AI execution.

### Scheduled Tasks
Cron-based recurring execution via platform-native schedulers (launchd on macOS, cron on Linux). Task definitions at `~/.claude/schedules.json`, logs at `~/.claude/logs/`.

### Remote Control
- **Telegram**: Full fleet control via bot commands (`/status`, `/agents`, `/start_agent`, `/stop_agent`, `/ask`, `/usage`, `/help`). Messages without command → Super Agent. Multi-user.
- **Slack**: Equivalent via Socket Mode (no public URL needed). @mentions, DMs, thread-aware.

### Vault
Persistent document storage: SQLite FTS5 full-text search, nested folders, cross-agent access, file attachments. MCP tools: create, read, update, delete, search, file attach, folder ops.

### Google Workspace
Integration via `gws` CLI (Google Workspace CLI) as MCP server. Covers Gmail, Drive, Sheets, Calendar, Docs (R/W), Slides, Tasks, Chat, People, Forms, Keep. 100+ specialized skills available.

### Skills & Plugins
One-click install from skills.sh and built-in marketplace. Categories: code intelligence (LSP for TS, Python, Rust, Go), external integrations (GitHub, GitLab, Jira, Figma, Slack, Vercel), dev workflows (commits, PR review).

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Framework | Next.js (App Router) | 16 |
| Frontend | React | 19 |
| Desktop | Electron | 33 |
| Styling | Tailwind CSS | 4 |
| State | Zustand | 5 |
| Animation | Framer Motion | 12 |
| Terminal | xterm.js + node-pty | 5 / 1.1 |
| Database | better-sqlite3 | 11 |
| MCP SDK | @modelcontextprotocol/sdk | 1.0 |
| Telegram | node-telegram-bot-api | 0.67 |
| Slack | @slack/bolt | 4.0 |
| Validation | Zod | 3.22 |
| Language | TypeScript | 5 |

## Configuration & Storage

All paths under `~/.dorothy/` and `~/.claude/`:
- App config: `~/.dorothy/app-settings.json`
- Persisted state: `agents.json`, `kanban-tasks.json`, `automations.json`, `processed-items.json`
- SQLite: `vault.db`
- Claude Code: `~/.claude/settings.json`, `~/.claude/schedules.json`, `~/.claude/logs/`

## Pricing

Free forever, open source, no account required.

## Notable Design Decisions

1. **MCP-first**: All programmatic control exposed through five MCP servers — any MCP client can drive the system
2. **node-pty isolation**: Genuine PTY sessions per agent, not simple subprocess spawning
3. **Content-hash deduplication**: Automations track processed items by hash
4. **Platform-native scheduling**: launchd/cron rather than in-process timers
5. **JIRA → Kanban → Agent closed loop**: External issues auto-create Kanban tasks, auto-assigned to capable agents
6. **Socket Mode for Slack**: No public URL required
7. **Persistent agent state**: Agents survive app restarts
