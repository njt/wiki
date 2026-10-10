---
url: https://github.com/reladraw/reladraw
title: "reladraw — a text language for diagrams where you say where things go"
author: Joe Walsh
date_fetched: 2026-10-10
date_published: 2026-10-10
topics:
  - developer-tools
  - agent-coding-workflow
---

reladraw (v0.16.0, Apache-2.0, TypeScript, zero runtime dependencies) is a text diagram language that sits deliberately between Mermaid and draw.io. Mermaid-style tools take your boxes and edges and decide where everything goes; drawing tools give you absolute coordinates at the cost of dragging things by hand. reladraw instead has the author *state the arrangement in words* — `right of app`, `level with queue`, `between web and worker` — and computes only the distances. "Nothing is ever chosen for you" is the design's core promise, and the tool refuses rather than guesses when the source is ambiguous.

The mechanism is a constraint solve, not a search: every placement becomes an inequality (`pos[to] >= pos[from] + weight`), and the whole diagram is solved by longest-path (Bellman-Ford with the comparison flipped) for the tightest arrangement satisfying all constraints. Gaps are minimums, never exact distances, which is what lets "put a node between two others" push them apart by exactly what it needs and let them close back up when it is deleted. Unsolvable arrangements are rejected with the offending placements named.

The repo ships a CLI (`reladraw diagram.reladraw -o diagram.svg`), a `<reladraw-diagram>` web component that renders inline SVG from source in the page, and a Claude-style agent skill installable via `npx skills add reladraw/reladraw` for Claude Code, Codex, Cursor, Copilot and others. The skill's argument is explicitly agent-first: an agent can't see the SVG it produced, so with auto-layout it is guessing, whereas with reladraw re-reading its own source tells it where everything landed. Ambiguity is refused with named errors, which becomes the agent's feedback loop.

The ~11.6K-line TypeScript implementation is hand-rolled end to end — lexer, parser, resolver, constraint solver, and a grid-based orthogonal edge router that finds shortest ways around boxes with turn charges breaking ties — with no graph layout library underneath. The syntax reference (SYNTAX.md) is unusually honest, with sections recording what is defective, unchecked, or undecided.
