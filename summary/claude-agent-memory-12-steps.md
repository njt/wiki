---
url: https://xcancel.com/i/article/2058156429559636069
title: "How to give your Claude agent a memory in 12 steps: from first setup to self-improving"
author: Codez (@0xCodez)
date_fetched: 2026-07-08
date_published: 2026-05-23
---

A practical walkthrough for giving Claude agents persistent memory that survives
across sessions and improves over time. The guide builds four layers of
increasing sophistication, from built-in features to Anthropic's Dreaming
research preview.

**Foundation (Layers 1–2).** Claude's built-in Chat Memory, rolled out March
2026, synthesizes preferences from conversation history roughly every 24 hours.
The author recommends seeding it explicitly rather than waiting — one deliberate
message lands immediately. Projects provide persistent instructions across chats
inside the workspace, but do not retain conversation history by default; that
gap is where people get burned.

**Persistent memory for coders (Layer 3).** A living memory file (CLAUDE.md in
Claude Code) gives the agent a structured record of preferences, decisions,
workarounds, and recurring mistakes. Auto-memory in Claude Code writes back
corrections automatically. The key discipline: save only what would change
future behavior. A memory file that stores everything is as useless as one that
stores nothing.

**Self-improving memory — Dreaming (Layer 4).** Shipped as a research preview in
May 2026, Dreaming is a scheduled background process that consolidates session
transcripts and existing memory into a new, reorganized memory store: duplicates
merged, stale entries replaced, new insights surfaced. It requires a Managed
Agents API key and gated access. The dream produces a separate output store so
it can never corrupt the original; the author stresses reviewing output before
swapping it in. Harvey, a legal-AI company, reported roughly a 6× increase in
agent task-completion rates after enabling Dreaming for legal-drafting
workflows.

The article closes with four common mistakes: treating Projects as memory,
bloating CLAUDE.md, storing everything with no filter, and auto-deploying dream
output without review.
