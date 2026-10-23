---
url: https://github.com/grahac/brainprint
title: "brainprint"
author: grahac
date_fetched: 2026-10-24
date_published: 2026 (repo)
topics:
  - personal-agents
  - agent-memory-and-context
---

brainprint is a Python CLI (~1,300 lines of pipeline code plus a Claude Code plugin, a Codex plugin, a Pi extension and a `/decide` skill) that mines your coding-agent session history — Claude Code, Codex and Pi — into a personal "decision brain": a model-written document of how *you* answer the choices your agents put to you. Two deliverables get installed back into your harnesses: a one-page advisory injected at session start, and a `/decide` command that reads the full 8,000-word brain and predicts what you would pick on a pending decision. Both are advice-only; neither grants the agent authority to act.

The pipeline is four stages — read, mine, score, install. Session JSONLs are parsed into (agent turn, human reply) pairs, scrubbed of secrets *before* truncation, sliced into 100-pair chunks, and mined in parallel into decision notes that are merge-reduced tree-style into three brain lengths. The genuinely unusual part is that it measures itself: a second brain is mined only from history older than 21 days and tested on your recent decisions, with held-out extraction — the miner writes three candidate pages, one half of recent decisions chooses the best page, the other half reports accuracy against chance and against the agent's own recommendation. Every stage caches to disk so a failed run resumes.

The work stays under `$HOME`, model calls go through your own agent CLIs (Pi's default model or Haiku for mining, Sonnet for scoring) in a throwaway temp directory with tools off, and a consent prompt says exactly what leaves the machine before anything is sent.
