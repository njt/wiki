---
title: "engineering-notebook"
url: https://github.com/prime-radiant-inc/engineering-notebook
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Engineering Notebook

CLI tool that automatically ingests Claude Code and Codex session transcripts, generates LLM-powered summaries, and provides a web interface for browsing an engineering journal.

## Key Features

- Ingests session files from Claude Code and Codex, parsing conversations while stripping tool calls and thinking blocks
- Groups sessions by date and project, then uses Claude to generate engineering journal entries
- Runs a web server with multiple browsable views

## Web UI Components
- Journal view with three-panel layout (date index, entries, transcripts)
- Projects view showing timelines sorted by recency
- Calendar/Gantt view displaying project activity across dates
- Full-text search across journal entries and transcripts
- Settings panel for configuration and iCal feed subscription
- Resume commands for picking up where you left off in sessions

## Description
"An automatic engineering diary that watches your AI coding sessions and distills them into a searchable, browsable narrative of what you built, what problems you hit, and what decisions you made."

## Technical Stack
- Bun (runtime, bundler, test runner, SQLite)
- Hono (web framework)
- Anthropic SDK (Claude Haiku for summarization)
- HTMX (interactive UI)
- TypeScript (99.9% of codebase)

168 stars, Apache 2.0 license.
