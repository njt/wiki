---
url: https://github.com/openclaw/acpx
title: "acpx — Headless CLI client for stateful Agent Client Protocol (ACP) sessions"
author: OpenClaw
date_fetched: 2026-05-14
date_published: null
---

# acpx

[![npm version](https://img.shields.io/npm/v/acpx.svg)](https://www.npmjs.com/package/acpx)
[![npm downloads](https://img.shields.io/npm/dm/acpx.svg)](https://www.npmjs.com/package/acpx)
[![CI](https://github.com/openclaw/acpx/actions/workflows/ci.yml/badge.svg)](https://github.com/openclaw/acpx/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> acpx is in alpha and the CLI/runtime interfaces are likely to change. Anything you build downstream of this might break until it stabilizes.

Your agents love acpx! They hate having to scrape characters from a PTY session.

`acpx` is a headless CLI client for the Agent Client Protocol (ACP), so AI agents and orchestrators can talk to coding agents over a structured protocol instead of PTY scraping.

One command surface for Pi, OpenClaw ACP, Codex, Claude, and other ACP-compatible agents. Built for agent-to-agent communication over the command line.

## Key Features

- **Persistent sessions**: multi-turn conversations that survive across invocations, scoped per repo
- **Named sessions**: run parallel workstreams in the same repo (`-s backend`, `-s frontend`)
- **Prompt queueing**: submit prompts while one is already running, they execute in order
- **Cooperative cancel command**: `cancel` sends ACP `session/cancel` via queue IPC without tearing down session state
- **Soft-close lifecycle**: close sessions without deleting history from disk
- **Queue owner TTL**: keep queue owners alive briefly for follow-up prompts (`--ttl`)
- **Fire-and-forget**: `--no-wait` queues a prompt and returns immediately
- **Graceful cancel**: `Ctrl+C` sends ACP `session/cancel` before force-kill fallback
- **Session controls**: `set-mode` and `set <key> <value>` for `session/set_mode` and `session/set_config_option`
- **Crash reconnect**: dead agent processes are detected and sessions are reloaded automatically
- **Prompt from file/stdin**: `--file <path>` or piped stdin for prompt content
- **Config files**: global + project JSON config with `acpx config show|init`
- **Session inspect/history**: `sessions show` and `sessions history --limit <n>`
- **Local status checks**: `status` reports running/idle/dead/no-session, pid, uptime, last prompt
- **Client methods**: stable `fs/*` and `terminal/*` handlers with permission controls and cwd sandboxing
- **Auth handshake**: stable `authenticate` support via env/config credentials
- **Structured output**: typed ACP messages (thinking, tool calls, diffs) instead of ANSI scraping
- **Any ACP agent**: built-in registry + `--agent` escape hatch for custom servers
- **One-shot mode**: `exec` for stateless fire-and-forget tasks
- **Experimental flows**: `flow run <file>` for TypeScript workflow modules over multiple prompts
- **Runtime-owned flow actions**: shell-backed action steps can prepare workspaces and other deterministic mechanics outside the agent turn
- **Flow workspace isolation**: `acp` nodes can target an explicit per-step cwd, so flows can keep agent work inside disposable worktrees

## Install

```bash
npm install -g acpx@latest
```

Or run without installing:

```bash
npx acpx@latest codex "fix the tests"
```

Session state lives in `~/.acpx/` either way.

## Built-in Agents

| Agent      | Wraps                        |
| ---------- | ---------------------------- |
| `pi`       | Pi Coding Agent              |
| `openclaw` | OpenClaw ACP bridge          |
| `codex`    | Codex CLI                    |
| `claude`   | Claude Code                  |
| `gemini`   | Gemini CLI                   |
| `cursor`   | Cursor CLI                   |
| `copilot`  | GitHub Copilot CLI           |
| `droid`    | Factory Droid                |
| `iflow`    | iFlow CLI                    |
| `kilocode` | Kilocode                     |
| `kimi`     | Kimi CLI                     |
| `kiro`     | Kiro CLI                     |
| `opencode` | OpenCode                     |
| `qoder`    | Qoder CLI                    |
| `qwen`     | Qwen Code                    |
| `trae`     | Trae CLI                     |

Use `--agent` as an escape hatch for custom ACP servers.

## Usage Examples

```bash
acpx codex sessions new                        # create a session
acpx codex 'fix the tests'                     # implicit prompt
acpx codex --no-wait 'draft test migration plan' # enqueue without waiting
acpx codex cancel                               # cooperative cancel
acpx codex -s api 'implement token pagination'  # named session
acpx --approve-all codex 'apply the patch and run tests'
acpx codex exec 'what does this repo do?'      # one-shot
```

## Flows

`acpx flow run <file>` executes a TypeScript flow module:

- `acp` steps keep model-shaped work in ACP
- `decision()` and `decisionEdge()` wrap constrained-choice ACP branching
- `action` steps handle deterministic mechanics like shell commands
- `compute` steps do local routing or shaping
- `checkpoint` steps pause for something outside the runtime

## Session Behavior

- Sessions scoped per repo, resolved by walking up from cwd to nearest git root
- Named sessions via `-s <name>` for parallel workstreams
- Prompt queue: if a prompt is running, new ones queue and drain in order
- Queue owner idle TTL (default 300s, configurable with `--ttl`)
- Crash detection and auto-resume: dead pids are respawned, sessions reloaded
- `exec` is always one-shot, no saved session

## Configuration

Reads config in order (later wins):
1. `~/.acpx/config.json` (global)
2. `<cwd>/.acpxrc.json` (project)
3. CLI flags (always win)

## Output Formats

- `text`: human-readable stream (default)
- `json`: NDJSON events for automation
- `quiet`: final assistant text only
- `--suppress-reads`: suppress read-file contents

## Stats

- 2.7k stars, 259 forks
- TypeScript 92.1%, JavaScript 7.8%, Shell 0.1%
- 351 commits
- MIT license
- Alpha status

## License

MIT
