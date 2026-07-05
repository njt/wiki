---
title: "Claude Code Cheat Sheet"
url: https://cc.storyfox.cz/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

Comprehensive reference guide for Claude Code v2.1.140 (updated May 12, 2026).

Key sections: keyboard shortcuts (Ctrl+C cancel, Shift+Tab cycle permissions, Alt+T toggle thinking, Alt+O fast mode), 30+ slash commands (/goal for completion targets with progress overlays, /recap for context summaries, /compact for token optimization), MCP server configuration (local/project/user scopes, HTTP/stdio/SSE transport, alwaysLoad, maxResultSizeChars up to 500K).

Permission modes: default (prompts), acceptEdits (auto-approve), plan (read-only), dontAsk (deny unless allowed), bypassPermissions (skip all).

CLAUDE.md files at four levels (project, local, personal, managed policy) with auto-loading of MEMORY.md (first 25KB or 200 lines). Rules importable via frontmatter paths.

Advanced workflows: batch operations auto-create git worktrees, voice mode in 20 languages, /btw for side questions without context costs, /loop for recurring tasks.

Settings cascade through user, project, and local JSON files. Environment variables for overrides. Managed policies via drop-in fragments.

Skills: /simplify, /batch, /debug, /claude-api. Custom skills in .claude/skills/ with frontmatter for description, allowed-tools, model override, dynamic context injection.