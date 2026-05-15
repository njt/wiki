---
title: "claude-replay"
url: https://github.com/es617/claude-replay
date_fetched: 2026-05-14
section: "Random"
---

# claude-replay: AI Session Replay Tool

Converts coding sessions from AI agents (Claude Code, Cursor, Codex, Gemini, OpenCode) into interactive, shareable HTML replays. Single self-contained files with no external dependencies.

Core capabilities:
- Auto-detects multiple transcript formats
- Standalone HTML files for blogs, documentation, demos
- Interactive playback with adjustable speed (0.5x to 5x)
- Collapsible tool calls and thinking blocks
- Built-in bookmarks/chapters system
- Multiple color themes with customization
- Terminal-style bottom-to-top scrolling
- File activity sidebar showing touched files
- Automatic secret redaction (API keys, tokens, connection strings)
- Live watch mode for monitoring agent sessions in real-time
- Web-based editor UI for visual session editing
- Chaining multiple sessions into one replay
- Embeddable via iframe

Supported sources: Claude Code (~/.claude/projects/), Cursor, Codex CLI, Gemini CLI, OpenCode.

Install: npm install -g claude-replay (or npx for zero install)

Architecture: Three-stage -- parsing JSONL transcripts, rendering with data compression (deflate + base64), vanilla JavaScript playback. Browser-native DecompressionStream API decompresses at load time. Compression typically reduces size 60-70%.

Privacy note: "Replay files embed the full session transcript, including source code, file paths, tool inputs/outputs." Users should review before public sharing and use --turns or --redact to exclude sensitive content.

681 stars, 41 forks, MIT licensed, 15 releases (latest v0.8.0).
