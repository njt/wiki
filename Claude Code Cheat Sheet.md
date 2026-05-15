# Claude Code Cheat Sheet

A comprehensive reference for Claude Code v2.1.140 (May 2026) covering keyboard shortcuts, slash commands, MCP configuration, permission modes, memory management, and advanced workflows. The fine print that saves hours when you need it.

---

## Key Themes

#devtools #agentic-coding

Highlights of what's documented:

**Keyboard shortcuts** -- Ctrl+C cancel, Shift+Tab cycle permission modes, Alt+T toggle extended thinking, Alt+O fast mode. Vim visual modes for text selection.

**Slash commands** -- 30+ commands. Notable: `/goal` sets completion targets with live progress overlays, `/recap` summarizes context, `/compact` optimizes token usage, `/btw` asks side questions without context cost, `/loop` runs recurring tasks.

**MCP configuration** -- servers at local/project/user scope, HTTP/stdio/SSE transport, `alwaysLoad` for persistent connections, `maxResultSizeChars` up to 500K per tool.

**Permission modes** -- default (prompts), acceptEdits (auto-approve), plan (read-only), dontAsk (deny unless allowed), bypassPermissions (skip all). Hooks support conditional logic with `continueOnBlock`.

**Memory hierarchy** -- CLAUDE.md at four levels (project, local, personal, managed policy), auto-loading MEMORY.md (first 25KB/200 lines), rules importable via frontmatter.

**Advanced workflows** -- batch operations auto-create git worktrees, voice mode in 20 languages, `/agents` shows unified session list, `/ultrareview` performs parallel multi-agent PR analysis.

**Custom skills** -- `.claude/skills/` with frontmatter for description, allowed-tools, model override, dynamic context injection. Agent frontmatter controls isolation, memory persistence, background execution, turn limits.

## Critical Analysis

This is the reference card that should ship with Claude Code but doesn't. The value is in the density: it covers features like `/btw` and `continueOnBlock` that aren't obvious from the docs. The cheat sheet format is perfect for a tool this feature-dense.

The risk of any comprehensive cheat sheet: it becomes outdated quickly. Claude Code ships updates frequently, and the May 2026 snapshot may not reflect current behavior. But as a starting point for discovering features you didn't know existed, it's excellent.

---
*Sources: [[raw/claude-code-cheat-sheet]]*
*Last updated: 2026-05-14*