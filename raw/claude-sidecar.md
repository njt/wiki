---
url: https://github.com/jrenaldi79/sidecar
title: Claude Sidecar
author: John Renaldi (jrenaldi79)
date_fetched: 2026-05-15
date_published: 2026-03-15 (v0.5.2)
---

# Claude Sidecar

A parallel AI window that runs alongside Claude Code or Claude Cowork. Shares your conversation context, lets you talk to other models (Gemini, GPT, DeepSeek, Qwen, Grok, etc.) simultaneously, then folds results back into your main session.

- **Author:** John Renaldi (GitHub: jrenaldi79)
- **Website:** claudesidecar.ai
- **License:** MIT
- **Stars:** 13 | **Forks:** 7 | **Open Issues:** 10
- **Language:** JavaScript (91.2%), HTML (7.8%), Shell (1.0%)
- **Latest Release:** v0.5.2 (March 15, 2026)
- **Total Releases:** 15 | **Commits:** 294

## What It Does

1. **Context Sharing** — Automatically reads your Claude Code session history from `~/.claude/projects/[project]/[session].jsonl` and passes it to the sidecar model. No copy-pasting needed.
2. **Parallel Interaction** — Main Claude session keeps working while you interact with the sidecar simultaneously.
3. **Multi-Model Support** — Connects to any model via OpenRouter or direct API keys. Ships with aliases for Gemini, GPT, Claude, DeepSeek, Qwen, Mistral, Grok, Kimi, GLM, MiniMax, and Seed.
4. **Fold Mechanism** — Click FOLD (or Cmd+Shift+F) to generate a structured summary that flows back into Claude Code context.
5. **Two Modes** — Interactive (Electron UI window) or Headless (`--no-ui` for autonomous background work).
6. **Agent Modes** — Chat (human-in-the-loop), Plan (read-only review), Build (full autonomy with write/bash access).
7. **MCP Integration** — Exposes tools (sidecar_start, sidecar_status, sidecar_read, etc.) as an MCP server, auto-registered for Claude Desktop and Cowork.
8. **Session Persistence** — List, read, resume, or chain sessions.
9. **Adaptive Personality** — Detects launch context (Claude Code vs. Cowork) and adjusts behavior accordingly.
10. **Safety Features** — Conflict detection, drift awareness, pre-flight validation.
11. **Auto-Update** — Checks npm registry daily; one-click update in UI.

## Architecture

Sidecar is a harness built on OpenCode (opencode.ai), which provides the conversation runtime, tool execution, agent system, and web UI. Sidecar adds context sharing from Claude Code, the Electron shell, fold/summary workflow, session persistence, and multi-client support.

The Electron shell uses a BrowserView architecture: OpenCode web UI loads in a dedicated viewport, toolbar renders in the bottom 40px.

## Installation

```bash
npm install -g claude-sidecar
sidecar setup  # graphical setup wizard for API keys, default model, routing
```

## Commands

- `sidecar start --model <model> --prompt "<task>"` — Launch a sidecar
- `sidecar list` — Browse past sessions
- `sidecar resume <task_id>` — Reopen a session
- `sidecar continue <task_id> --prompt "..."` — New session building on previous
- `sidecar read <task_id>` — Read session output
- `sidecar abort <task_id>` — Stop a running session
- `sidecar setup` — Configure API keys, models, routing
- `sidecar update` — Update to latest version

## Model Aliases

| Alias | Model |
|-------|-------|
| `gemini` | Gemini 3.1 Flash Lite |
| `gemini-pro` | Gemini 3.1 Pro (1M context) |
| `gpt` | GPT-5.4 |
| `codex` | GPT-5.3 Codex |
| `claude` / `sonnet` | Claude Sonnet 4.6 |
| `opus` | Claude Opus 4.6 |
| `deepseek` | DeepSeek v3.2 |
| `qwen` | Qwen 3.5 397B |
| `grok` | Grok 4.1 Fast |

## MCP Tools

| Tool | Description |
|------|-------------|
| `sidecar_start` | Spawn a sidecar (returns task ID immediately) |
| `sidecar_status` | Poll for completion |
| `sidecar_read` | Get results |
| `sidecar_list` | List past sessions |
| `sidecar_resume` | Reopen a session |
| `sidecar_continue` | New session building on previous |
| `sidecar_abort` | Stop a running session |
| `sidecar_setup` | Open setup wizard |
| `sidecar_guide` | Get usage instructions |

## Use Cases

- Fact-check Claude's work
- Get a second opinion on a feature
- Deep-dive without polluting context
- Parallel investigation (implement + review simultaneously)
- Leverage model strengths (Gemini for 1M context, GPT for code gen, DeepSeek for cost-effective reasoning)
