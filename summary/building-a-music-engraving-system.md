---
url: https://noteuniverse.com/blog/building-a-music-engraving-system/
title: "Building a Music Engraving System"
author: Note Universe author (unnamed in post)
date_fetched: 2026-11-09
date_published: 2026 (recent, exact date not stated)
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

A solo developer's two-year build of a music engraving engine — the layout-and-rendering pipeline behind Note Universe, a sight-reading practice app. The post is a craft write-up: decoupling engraving from rendering, a "Renderables" abstraction that makes the renderer dumb, imperative versus constraint-based layout, ASCII-fixture unit tests and 200+ screenshot goldens, and the switch to the free Bravura font under SMuFL scaling.

The author hand-wrote the engine and app for the first year, then since roughly November 2025 has done all ongoing development through Claude Code, keeping the human-written text of the article as an explicit provenance statement. The architecture paid off when AI entered the picture: refactoring the engraver API to be general enough for both procedural exercise generators and an external importer took little work, and the MusicXML parser connected to it easily.

The payoff is agentic: with the parser in place, LLM-generated MusicXML becomes a content pipeline. Asking Claude Fable to compose a short waltz in G major, importing it, and sight-reading it on a tablet closes the loop — the AI-generated exercises the author predicted are now shipping alongside the procedural ones.
