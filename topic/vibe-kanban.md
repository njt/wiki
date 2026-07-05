# vibe-kanban

Kanban boards for managing AI coding agents: plan work, assign it to agents (Claude Code, Codex, Gemini CLI, Copilot, Cursor, and more), review the generated code with inline diffs, and create PRs. Each agent gets its own workspace with a branch, terminal, and dev server. Already sunsetting -- the annotation notes "where's the money in this?" which is the right question.

---

## Key Quotes

> "Vibe Kanban is sunsetting."

## Key Themes

#coding-agents #project-management #tools #dev-tools #sunset

The concept is sound: as coding agents multiply, you need project management infrastructure specifically designed for them. Traditional kanban boards assume human workers; vibe-kanban assumes AI agents that need isolated workspaces, automated code review, and PR generation. This is the management layer for the multi-agent coding systems described in [[Scaling Long-Running Agents]].

Supporting 10+ coding agents is both a feature and a symptom. The coding agent market is so fragmented that a management tool needs to support Claude Code, Codex, Gemini CLI, Copilot, Amp, Cursor, and more. That fragmentation makes the management layer valuable but also makes it hard to build deep integrations with any single agent.

## Critical Analysis

The sunsetting tells the story. The tool solves a real problem (managing agent-generated code) but the problem isn't valuable enough to sustain a business. Developers who use coding agents are comfortable reviewing code in their existing tools (VS Code, GitHub PRs). Adding a separate kanban layer on top is overhead rather than simplification.

The Rust + TypeScript stack and the rapid feature development (inline diffs, browser preview, device emulation) suggest solid engineering. The failure was market, not technology.

The deeper lesson: tooling for managing AI agents is a thin layer on top of existing workflows. The value accrues to the agents themselves (Claude Code, Cursor) and to the platforms (GitHub, linear), not to middleware between them. This connects to Hightouch's observation in [[How Hightouch Built Their Long-Running Agent Harness]] that the real work is context engineering, not orchestration tooling.

---
*Sources: [[summary/vibe-kanban]]*
*Last updated: 2026-05-14*
