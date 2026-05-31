---
url: https://broomy.org/
title: Broomy — Command Center for AI Coding Agents
author: Broomy (broomy-ai)
date_fetched: 2026-05-31
date_published: unknown
---

# Broomy — Command Center for AI Coding Agents

## Source Content (fetched 2026-05-31 from broomy.org)

### What Is It?

Broomy is a free, open-source (MIT-licensed) native desktop app built with TypeScript, React, and Electron. It serves as a unified window for managing multiple AI coding agents simultaneously.

### Who Made It?

The project lives at github.com/broomy-ai/broomy under the Broomy organization. No individual creators are named on the page.

### What Problem Does It Solve?

AI coding agents create chaos: "Terminal tabs everywhere. Which agent finished? Which one needs input? What branch is it on? What files did it change?" Broomy consolidates all of that into a single window with visibility into each agent's status.

### How It Works

It runs terminal-based coding agents (like Claude Code, Aider, Codex, Gemini CLI — or any other) side by side. Broomy monitors each session, displaying who is working, who is idle, and who needs attention — with notifications when tasks complete.

### Key Features

1. **Multi-agent session management** — Run different agents on different parts of a project (e.g., "Claude Code on your backend, Aider on your frontend") all in one window.

2. **AI-guided code review** — Uses AI to summarize changes, highlight potential issues, and link to diffs. The page notes it's "not about rubber-stamping AI output" but about keeping humans in the loop so the codebase stays healthy.

3. **Built-in IDE** — File editing, repo browsing, git status, staging, and committing — no need to switch windows. The philosophy: "You're not forced to hand everything to the AI."

4. **Agent-agnostic** — Works with any terminal-based coding agent, and adding new ones is described as trivial.

5. **Local-only** — "No cloud. No accounts. Your code and sessions stay on your machine."

### Design Philosophy & Interesting Aspects

- **Actually open source**: "No telemetry. No paid tier. No 'open core' bait-and-switch." Every line is on GitHub to read, fork, change, or ship a custom version.
- **Extensibility emphasized**: Users are encouraged to fork and modify or contribute back via issues, PRs, or discussions.
- **Current status**: Public Preview — "stable enough for daily use, but you may run into issues."
- **Platform**: Mac-only release currently; Windows and Linux are "coming soon."
- **Tech stack**: Electron + React + TypeScript. Setup requires pnpm and Node.js — clone the repo, run `pnpm install && pnpm dev`.

### Relevant Quotes from the Landing Page

> "Terminal tabs everywhere. Which agent finished? Which one needs input? What branch is it on? What files did it change?"

> "Not about rubber-stamping AI output" — on the code review feature

> "You're not forced to hand everything to the AI."

> "No cloud. No accounts. Your code and sessions stay on your machine."

> "No telemetry. No paid tier. No 'open core' bait-and-switch."

GitHub: https://github.com/broomy-ai/broomy
