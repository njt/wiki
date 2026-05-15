# claude-replay

Converts AI coding agent sessions (Claude Code, Cursor, Codex, Gemini, OpenCode) into self-contained, embeddable HTML replays. Think screen recordings but for agent sessions -- interactive, searchable, and vastly smaller than video files. A single HTML file with no external dependencies.

---

## Key Quotes

> "Replay files embed the full session transcript, including source code, file paths, tool inputs/outputs."

## Key Themes

#tool #agent-observability #replay #documentation

This fills a genuine gap in the agent tooling ecosystem. Agent sessions produce JSONL transcripts that are functionally unreadable. claude-replay turns them into something you can share in a blog post, attach to a bug report, or use as teaching material. The fact that it supports five different agent formats (Claude Code, Cursor, Codex, Gemini, OpenCode) makes it a universal tool rather than a Claude-specific one.

The architecture is pragmatic: parse JSONL, compress with deflate, embed in vanilla JavaScript that decompresses via the browser's native DecompressionStream API. No React, no build system, no runtime dependencies. Size reduction of 60-70% means these are genuinely shareable.

The live watch mode is interesting for a different use case: monitoring running agent sessions in real-time. Combined with the web editor for visual session editing, this starts to look like a proper observability tool rather than just a replay generator.

## Critical Analysis

The automatic secret redaction (API keys, AWS credentials, bearer tokens) is essential but explicitly "best-effort." This is honest but means you should never blindly share a replay without reviewing it first. The tool itself warns about this.

The bookmark/chapters system addresses a real problem -- agent sessions can be very long, and navigating to the interesting part is important. But the bookmarks are manually added via `--mark "N:Label"`. What you really want is automatic detection of significant moments (test failures, tool errors, mode switches).

See also [[session-analysis]] for Leonard Lin's approach to understanding session dynamics, and [[agent-pr-replay]] for comparing agent vs human approaches to the same PR.

---
*Sources: [[raw/claude-replay]]*
*Last updated: 2026-05-14*
