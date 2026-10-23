---
url: https://html.moldandyeast.com/?open=shop
title: "Shop — html.moldandyeast.com (how it's built)"
author: html.moldandyeast.com (site archive; author unnamed)
date_fetched: 2026-10-24
date_published: 2026-10-21
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

This is not an article but a structural archive of **Shop**, an artifact in the html.moldandyeast.com library of single-file, dependency-free HTML artifacts — a "design canvas built on code, where you and AI agents design side by side and every edit stays real, reviewable code." The page assembles the site's llms.txt, the index entry, the CSS/JS comments in file order, and two example preset programs into a readable anatomy of a 696 KB HTML file.

## The site and the artifact

The library's premise is striking: no prompts, just finished artifacts handed to an agent as a reference — "like this, but…". Shop's index entry (added 2026-10-21) is the canvas-under-cursor idea taken to its limit. The 696 KB file contains two inline module scripts (145 KB and 385 KB), 71 KB of CSS, and 35 text/plain preset programs. The site exposes a read-only, no-auth MCP server (search_artifacts, get_artifact, list_tags) plus a JSON search index.

## The architecture, via comments

The comment trail describes a real design-tool stack in miniature:

- **Trellis core** — grammar → IR → DAG → runtime. One builder node `N`, everything a fluent chain; any value may be a lambda, lambdas reading state stay live at runtime, the rest fold at build time.
- **Compile** — design + parameters → flat IR node table + DAG, with dependency tracking through proxies.
- **Renderer** — IR → live DOM, "the canvas is real HTML/CSS, like Paper" — with meticulous accommodations for the host canvas's quirks (whole-pixel text widths, top-left rotation origins, whitespace collapsing).
- **Exporters** — Paper HTML, standalone HTML, JSX, IR JSON, and builder code ("eject").
- **Canvas edits → code** — edits made on the canvas are written back into the program as `edits({...})` entries keyed to stable paths, applied in rounds, with careful handling of renames, moves, and sibling collisions ("'Row' never makes its sibling 'Row#2' answer to the first one's edits").
- **SUL and the Lab** — a semantic UI language (one line per element, indentation nests, every prefix is a valid program so it renders while streaming) styled by a learned 16-number taste vector θ, learned from A/B picks via a hand-wired ReGLU preference model with five bootstrap-resampled scorers whose disagreement is treated as honest doubt.
- **Co-design** — agents work on the canvas beside the person: tiered reads, all-or-nothing code-mode writes, reviewable turns the person can reject without losing their own edits, pins anchored to layers, protection and notes living in the program, and undo/redo that keeps turns honest.
- **Projects** — every design is one file, local-first, mirrored to the artifact's database, with automatic restore points before every agent turn.

## The two example presets

The two included programs show the grammar: a fluent-chained "Hello, frame" card, and a tokens-and-mixins profile card where tokens become CSS variables in the Tailwind-v4/Paper namespaces and mixins are plain functions applied with `.with()`.

## Take

Shop is a proof-by-artifact that a full agent-co-designing design tool — IR, canvas, learned taste model, review gates, persistence — fits in a single hand-auditable HTML file. The comments read as engineering craft notes, and the whole thing is itself the library's pitch: hand it to your agent and say "like this, but…".
