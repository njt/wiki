---
url: https://github.com/cathrynlavery/diagram-design
title: "Diagram Design"
author: Cathryn Lavery
date_fetched: 2026-09-13
date_published: 2026-09-10
topics:
  - claude-code
  - developer-tools
---

Diagram Design is Cathryn Lavery's Agent Skill (v2.6.x) for Claude Code, Codex, GitHub Copilot, Factory Droid, Pi, Kiro, and OpenCode that produces 40 editorial diagram types — architecture, sequence, Sankey, Wardley map, waterfall, and more — as single self-contained HTML files with inline SVG. No build step, no JavaScript by default, no Mermaid-renderer output: the skill gives the model an opinionated design system (jet-black + atomic-tangerine palette, Instrument Serif/Geist/Geist Mono type stack, 1px hairlines, a 4px coordinate grid, 1–2 focal accents, ≤9 nodes per diagram) and enforces it with deterministic CI gates rather than hoping the model has taste.

The architecture is progressive disclosure all the way down. A ≤40 KB `SKILL.md` (ADR 0004) routes each request to exactly one `references/type-*.md` layout grammar; eight semantic patterns describe behavior (queues, policy traces, trust boundaries) and always route to the nearest existing visual type, never a new one (ADR 0002). `references/style-guide.md` is the single token source, onboarded from your website's palette/fonts in ~60 seconds, and shared across clients via named profiles in `~/.diagram-design/profiles/` selected by a per-project `.diagram-design` marker. Importers for draw.io, Mermaid, and Excalidraw extract a structured IR with deterministic Python scripts, then the model *redraws* — never converts — at a chosen format, size, detail, and audience, reporting a fidelity ledger of what was merged or dropped.

What makes it notable as an artifact is the verification discipline: ~40 `verify-*.py` gates in CI check label-mask geometry, treemap area conservation, Sankey flow conservation, waterfall running totals, a byte-pinned motion controller, and rendered clipping measured by pixel-diffing headless-Chromium screenshots; a distilled `self_check.py` ships to installed agents so the same contract applies to generated output. Motion is optional, static by default, and fully accessible (prefixed `<title>`/`<desc>` IDs, `prefers-reduced-motion` support). It is one of the most rigorously engineered examples of a cross-host Agent Skill.
