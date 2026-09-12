---
url: https://www.npmjs.com/package/jouzu
title: jouzu
author: Shisa AI
date_fetched: 2026-09-08
date_published: undated (alpha v0.1.8)
topics:
  - coding-agents-and-frameworks
---

# jouzu

Jouzu is Shisa AI's agentic coding harness, built on the Pi coding agent. It repackages Pi with the specific tools and workflows Shisa uses daily, shipping them "batteries included": goal tracking and measured loops, background shell jobs with batched completion summaries, child agents with assignable roles, browser-backed web search, local content scanning, searchable session history, CJK-safe terminal layout, voice dictation, and a model palette that remembers per-model preferences.

It is not a fork — Jouzu pins Pi as an exact dependency and owns its update path (Pi's own self-update is blocked). On top it layers a curated extension set (`schedule_prompt`, `bg_task`, `web_fetch`/`tff-*`, task and goal tools, `pi-vcc` compaction with `vcc_recall`), a two-profile system (`core` as the safe, language-neutral fallback and an optional `ja` Japanese profile), an isolated state root, and an unusually rigorous self-updater that packs the current install as a rollback artifact and verifies SHA-512 before relaunching.

The Shisa connection is commercial but opt-in: a built-in `shisa-api` model-catalog source activates only when `SHISA_API_KEY` is present, and voice dictation routes audio to Shisa's realtime ASR, never auto-sends, and writes no recording files. Consent is engineered carefully throughout — non-interactive first runs default to `core` and "do not manufacture consent," and Pi state is imported only after an explicit per-file prompt that defaults to no. The project is explicit that v0.1.x is alpha software used as the team's daily driver, with frequent updates expected.
