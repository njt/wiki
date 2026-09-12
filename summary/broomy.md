---
url: https://broomy.org/
title: "Broomy — Your command center for AI coding agents"
author: broomy-ai
date_fetched: 2026-05-31
date_published: unknown
topics:
  - developer-tools
---

Broomy is an MIT-licensed Electron desktop app (TypeScript + React) that runs multiple terminal-based AI coding agents side-by-side in a single window. It combines a session dashboard, AI-guided code review, and a built-in IDE (file editing, git). Mac-only Public Preview. Works with Claude Code, Codex, Gemini CLI, and any terminal-based agent.

## Landing Page Content (surf grab)

**Tagline:** Your command center for AI coding agents. Lead a team of agents and see when each needs help. Let an AI help you review code. Fully open source and extensible.

**Status:** Public Preview — stable enough for daily use, but you may run into issues. MIT License. TypeScript + React + Electron.

**Problem statement:** If you use AI coding agents, you know the pain. Terminal tabs everywhere. Which agent finished? Which one needs input? What branch is it on? What files did it change? Broomy gives you a single window to manage all of it — and since it's open source, you can extend it to fit exactly how you work.

### Sessions
Run many agents at once. Run Claude Code on your backend, Aider on your frontend, and keep a terminal open for your docs — all in one window. Broomy watches each agent and shows you who's working, who's idle, and who just finished and needs your attention. When an agent completes a task, you get a notification so nothing slips through.

### Review
AI-guided code review. AI agents write code fast — but someone still needs to review it. Broomy uses AI to help you understand what changed and why, highlights potential issues, and lets you click through to diffs. It's not about rubber-stamping AI output — it's about keeping you in the loop so the codebase stays healthy even as agents move fast.

### IDE
A real IDE, not just a chat window. Edit files, browse your repo, check git status, stage changes, and commit — without switching windows. You're not forced to hand everything to the AI. When you need to step in and fix something yourself, the tools are right there.

### Agents
Works with every agent. Broomy doesn't lock you into one AI provider. It works with any terminal-based coding agent, and adding support for new ones is trivial. Claude Code, Codex, Gemini CLI, + any agent.

### Built to be extended
Broomy is a native desktop app built on well-understood technology. The codebase is designed to be readable, hackable, and easy to contribute to.

### Actually open source
MIT licensed. No telemetry. No paid tier. No "open core" bait-and-switch. Every line of code is on GitHub — read it, fork it, change it, ship your own version.

- Fork & modify: Don't like something? Change it. Add your own panels, agents, or workflows.
- Contribute back: Open an issue, send a PR, or start a discussion. The project grows from community input.
- Runs locally: No cloud. No accounts. Your code and sessions stay on your machine.

### Get started
```bash
git clone https://github.com/broomy-ai/broomy.git
cd broomy
pnpm install
pnpm dev
```
Requires pnpm and Node.js. Also available as pre-built Mac download. Windows and Linux coming soon.

Source: https://broomy.org/ (fetched via surf browser automation)
