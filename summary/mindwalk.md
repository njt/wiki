---
url: https://github.com/cosmtrek/mindwalk
title: mindwalk
author: Ricko Yu (cosmtrek)
date_fetched: 2026-07-18
date_published: 2026
topics:
  - developer-tools
---

mindwalk replays coding-agent sessions on a 3D map of your codebase. It draws the
repository as a night citymap and plays a session back as light moving through it —
files the agent searched, read, or edited glow; everything else stays dark. A single
Go binary reads Claude Code and Codex session logs, renders the 3D frontend via an
embedded React + Three.js app, and never sends data off the machine.

The pipeline produces three independent artifacts. A **trace** normalizes session
logs into an ordered stream of file-touch events classified as search, read, edit,
exec, verify, or other. A **citymap** lays out the file tree using a deterministic
squarified treemap so that the same repository always produces the same visual
layout, making cross-session comparisons meaningful. A **report** runs a sealed LLM
judge against a synthesized evidence document, producing qualitative findings whose
severities are mechanically rolled up into dimension verdicts.

The tool classifier maps ~30 agent tools to actions, parsing shell pipelines to
distinguish `grep` (search) from `cat` (read) from `npm test` (verify). Weak target
tracking marks paths extracted from command strings rather than structured inputs,
filtering them against actual filesystem state. The agent graph system reconstructs
parent-child relationships across subagent sessions with explicit link quality tiers:
exact, derived, or unavailable.

Computed stats include a re-read regression rate (repeated reads without intervening
edits, a proxy for agent wandering), churn detection (files edited 3+ times, edits
after the last verify), and an observability grade per derived stat — exact,
estimated, or unavailable — that feeds the verdict rollup. Ghost files (touched by
the session but no longer on disk) render as wireframe outlines, telling a visual
story about sessions that operated on stale data.

The architecture is deliberately zero-dependency: one Go binary, `//go:embed` for
the frontend, no database, fingerprint caching everywhere. Viewing is fully local;
the LLM judge is opt-in and runs through the user's own CLI with their own API key.
---
*Sources: [[raw/mindwalk]]*
*Last updated: 2026-08-01*
