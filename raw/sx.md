---
url: https://github.com/sleuth-io/sx
title: "sx: Your team's private npm for AI assets"
author: sleuth-io
date_fetched: 2026-05-16
date_published: 2025
---

# sx — Package Manager for AI Coding Assistants

sx is a CLI tool from sleuth-io that acts as "your team's private npm for AI assets" — skills, MCP configs, commands, rules, agents, hooks, and plugins for AI coding assistants. Written in Go, Apache 2.0 licensed, 143 stars, 53 releases (latest v1.0.4, May 15, 2026).

## Problem

Expert AI coding assistant users develop custom configurations (skills, MCP configs, slash commands, patterns), but that knowledge remains siloed on individual machines. Existing workarounds — repo-by-repo copying, global config bloat, client-specific plugins — don't scale across teams.

## Asset Types

- **Skills** — custom prompts/behaviors for specific tasks
- **Rules** — coding standards tied to file types/paths
- **Agents** — autonomous AI agents with defined goals
- **Commands** — slash commands for quick actions
- **Hooks** — lifecycle automation triggers
- **MCP Servers** (experimental) — Model Context Protocol integrations
- **Plugins** — Claude Code plugin bundles combining multiple asset types

## Installation Scoping

Assets can be scoped to specific targets: `--org` (entire vault, default), `--repo`/`--path` (specific repo or subpath), `--team` (admin-gated team members), `--user` (single human, must match caller's git identity), `--bot` (CI runner/agent, requires `SX_BOT=<name>`).

## Distribution Models

1. **Local (Personal)** — path-based vault for personal project sharing
2. **Git vault (Small teams)** — shared git repo vault
3. **Skills.new (Large teams/enterprise)** — centralized UI with analytics

## Architecture

Follows the manifest-and-lock pattern (npm, cargo, uv):
- **Manifest (`sx.toml`)** — vault source of truth listing every managed asset, install scopes, and team definitions
- **Lock file** — per-user resolved artifact, stored in `~/<cache>/sx/lockfiles/`, previous versions rotated with timestamps
- **Audit + usage streams** — mutations append to `.sx/audit/YYYY-MM.jsonl`; usage events append to `.sx/usage/YYYY-MM.jsonl`

## Supported Clients

Claude Code, Cline, Codex, Cursor, GitHub Copilot, Gemini (CLI/VS Code/JetBrains/Android Studio), Kiro, Openclaw — plus claude.ai and chatgpt.com via cloud relay.

## Cloud Relay

Exposes vaults as MCP endpoints for web-based AI assistants. Relay forwards requests over a WebSocket the user's machine opens — vault content stays local. Commands: `sx cloud connect`, `sx cloud serve`, `sx cloud status`.

## skills.sh Integration

Integrates with skills.sh (85k+ community skills). Commands: `sx add anthropics/skills/frontend-design`, `sx add --browse` for directory searching.

## Analytics & Audit

- `sx stats` — adoption dashboard with `--since 7d --json` support
- `sx audit` — recent team/install mutations, filterable by actor, time range, event type

## Installation

- Homebrew: `brew tap sleuth-io/tap && brew install sx`
- Shell: `curl -fsSL https://raw.githubusercontent.com/sleuth-io/sx/main/install.sh | bash`

## Quickstart

`sx init` → `sx add /path/to/my-skill` → `sx install`

Multiple vaults via profiles: `sx profile add work`, `sx profile use work`, `sx profile list`.

Dry-run preview: `sx install --dry-run`.

## Roadmap

Remaining: RBAC and change request flow for gated skill update process.
