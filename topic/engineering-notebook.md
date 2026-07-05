# engineering-notebook

CLI tool from Prime Radiant that ingests Claude Code and Codex sessions, generates LLM-powered summaries, and serves a browsable engineering journal. An "automatic engineering diary" that watches your AI coding sessions and distills them into a searchable, browsable narrative of what you built, what problems you hit, and what decisions you made.

---

## Key Quotes

> "An automatic engineering diary that watches your AI coding sessions and distills them into a searchable, browsable narrative of what you built, what problems you hit, and what decisions you made."

## Key Themes

#developer-tools #session-analysis #engineering-journal #observability

The tool addresses a real problem: AI coding sessions are ephemeral by default. You do a four-hour session, ship the code, and the reasoning behind your decisions evaporates. engineering-notebook makes that reasoning durable.

The web UI is thoughtfully designed -- three-panel journal view, project timelines, calendar/Gantt view of activity, full-text search, and resume commands. The iCal feed subscription is a nice touch for integrating with existing workflows.

Built on Bun + Hono + HTMX + Claude Haiku, which is a pleasingly lightweight stack for a tool that could easily have been over-engineered.

Connects to [[napkin]] (both are about making agent sessions leave traces) and [[Three Tier Memory]] (which addresses the same problem at the architecture level rather than the tooling level). Also complements [[Cognitive Debt]] -- if cognitive debt is about the gap between production and comprehension, engineering-notebook is one attempt to close that gap by making the production process legible after the fact.

## Critical Analysis

The main risk is that this becomes write-only: the journal fills up but nobody reads it. The value depends entirely on whether developers actually consult past entries when starting new work. The search and resume features help, but the tool would benefit from proactive surfacing -- "you worked on something similar three weeks ago, here's what happened." Still, even a write-only journal has forensic value when something breaks six months later.

---
*Sources: [[summary/engineering-notebook]]*
*Last updated: 2026-05-14*
