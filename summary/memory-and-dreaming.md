---
url: https://devin.ai/blog/memory-and-dreaming
title: "Memory and Dreaming in Devin"
author: Cognition (Devin team)
date_fetched: 2026-10-08
date_published: 2026-10-08
topics:
  - agent-memory-and-context
  - coding-agents-and-frameworks
---

Cognition announces Memory and Dreaming for Devin. **Memory** is a persistent, personal store of "lessons Devin learned from working with you" — preferences, corrections, project gotchas — implemented as a Git repository of markdown notes (a "Memory Drive") with a short `MEMORY.md` index loaded at session start; the agent searches and reads the rest with its normal code-navigation tools rather than loading everything into context. **Dreaming** is a daily asynchronous background session that consolidates overlapping notes, strips transient detail, mines un-captured lessons, and deletes stale records, while preserving source links and explicit preferences.

Key implementation details: each session works on its own Git checkout of the drive, commits changes, merges concurrent updates, and a revision check rejects stale writes with conflicts surfaced for resolution rather than silently overwritten — parallel sessions converge on one memory store without any single session's copy being authoritative. Memories are explicitly distinguished from skills: skills package repeatable procedures for reuse, memory accumulates contextual lessons with a different lifecycle. Memories are personal to a user within an organization, not shared team instructions. Cognition also open-sourced the memory standard at cognition.ai/agent-memory-repo.

The post is short and product-announcement-shaped, but its architecture choices — git-as-memory-store, index-first retrieval, offline consolidation, explicit memory/skill split — are a concise statement of where production agent memory has landed.
