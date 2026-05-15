# AgentsView

A local-first analytics dashboard for AI coding agent sessions. One Go binary, no accounts, everything local. Auto-discovers sessions from 24+ agents (Claude Code, Codex, Cursor, OpenHands, VSCode Copilot, Gemini CLI), provides full-text search across message content, token/cost tracking with prompt-caching-aware calculations, and activity analytics. Claims 100x faster queries than similar tools via indexed SQLite.

---

## Key Themes

#developer-tools #analytics #cost-tracking #privacy #local-first

This fills a real gap: if you're running multiple coding agents, you need visibility into what they're doing and what they cost. The privacy stance (no telemetry, no accounts, localhost-only by default) is appropriate for a tool that reads your agent conversations.

The cost tracking with prompt-caching awareness is a practical feature -- most cost estimators ignore caching, which means they overcount by 2-5x for heavy users. The optional PostgreSQL backend for team dashboards is a clean upgrade path from solo to shared use.

Connects to [[Dorothy]] (which provides live orchestration rather than post-hoc analytics) and the broader observability concern in [[What I learned building an opinionated and minimal coding agent]] (Zechner's argument for full visibility into agent execution).

## Critical Analysis

Strong: auto-discovery of 24+ agent session formats is impressive plumbing work. FTS5-based search across all your agent conversations is genuinely useful for "what did I try last week?" queries. The Go + Svelte + SQLite stack is the right choice for a local tool: fast, small, no runtime dependencies.

Weak: read-only analytics means you can't act on what you find -- no way to replay, branch, or resume sessions from within AgentsView. The Tauri desktop wrapper adds Electron-like overhead for what could be a simpler CLI + browser UI. The "100x faster" claim lacks a benchmark methodology.

The session archetype and velocity metrics in `agentsview stats` could be genuinely interesting for understanding your own coding patterns -- how much of your work is agent-driven vs. manual, which models you reach for, what time of day you're most productive with agents.

---
*Sources: [[raw/agentsview]]*
*Last updated: 2026-05-14*
