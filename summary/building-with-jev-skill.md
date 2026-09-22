---
url: https://github.com/dbreunig/building-with-jev-skill
title: "Building with Jev"
author: Drew Breunig
date_fetched: 2026-09-22
date_published: 2026-09-17
topics:
  - claude-code
  - agent-architecture
---

Drew Breunig's repo ships a single agent skill — `skills/jev/SKILL.md`, 213 lines of pure prose with plugin and marketplace manifests beside it — that teaches a coding agent how to write and debug programs calling Jev, TypeSafe's "System One" judgment model. One commit (2026-09-17), version 0.1.0, no code, no tests: the artifact is the knowledge, and the host agent is the runtime.

The skill's core move is a strict division of labor. Jev reads one structured `state`, answers every question in a request independently and in parallel, and returns probability distributions over answer sets you predefined; it never generates text, never reasons in steps, and never counts. The program owns control flow, arithmetic, and policy; the model owns "snap judgments" — questions a knowledgeable person answers in a second, given the right context. Three primitives cover the surface: Choice (a branch per option), Score (a threshold, rank, or weight on a described spectrum), and Noul (a yes/no whose distance from 0.5 is its own confidence).

Around that core the skill packages practitioner discipline: atomic one-property questions with backticked state paths, contrastive criteria (`what`/`not_for`/`examples`), situation-described Score levels that stand alone, a minimal state that converts numbers to words and does arithmetic in code, speculative fan-out of every possibly-needed question in one parallel request, confidence-gated routing with risk-scaled thresholds (0.5–0.6 floors, 0.85–0.9 for high-stakes actions), and composite scoring that normalizes each dimension and combines with weights in code. It closes with a thirteen-row symptom→cause→fix diagnosis table, revision rules (change one or two questions at a time, judge on labeled data, keep the answer space stable once code depends on it), and a twelve-item checklist. The guidance is explicitly version-pinned to `jev-1.13`, with a pointer to recheck the vendor's "jaggedness" page when the model changes.

Distribution is threefold: a Claude Code plugin marketplace (`/plugin marketplace add dbreunig/building-with-jev-skill`, then `/plugin install jev@building-with-jev`), the vercel-labs skills CLI (`npx skills add`) for Claude Code, Codex, Cursor, and others, and a manual copy into `~/.claude/skills/jev`. The skill self-loads when a task involves Jev or TypeSafe questions, or on explicit `/jev` invocation.
