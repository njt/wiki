---
url: https://news.ycombinator.com/item?id=46515696
title: "HN: Opus 4.5 is not the normal AI agent experience that I have had thus far"
author: tbassetto (via burkeholland.github.io)
date_fetched: 2026-05-15
date_published: ~2026-01 (4 months before fetch)
topics:
  - agent-coding-workflow
---

Hacker News discussion (879 points, 1,353 comments) on a blog post by tbassetto describing their experience building applications with Claude Code + Opus 4.5. The OP created multiple apps (image converter, SVG editor, etc.) without understanding how they were assembled, claiming Opus 4.5 represents a qualitatively different tier of coding agent.

The thread is a Rorschach test for the state of AI-assisted development in early 2026. Comments range from "I don't write Rust code anymore, I just read it" (rtfeldman, Zed) to "on novel work it's a hindrance" (JDye, proxy servers) to "it's a better programmer than I am, it's just not anywhere near as good a software developer" (jaggederest).

Key themes that emerge:

**Compiler-as-guardrail**: Strongly typed languages (Rust, C#) give agents faster feedback loops via compile errors. Multiple commenters report better results with Rust than Python/JS because the compiler catches mistakes instantly. gck1 switched to Rust for everything, even throwaway scripts. "javascript can still have ugly runtime WTFs" (tezza).

**Training-data proximity**: LLMs excel at well-trodden patterns (React/TypeScript) but struggle with novel or low-training-data domains. C++ 3D graphics, MASQUE proxying protocols, and bleeding-edge Swift all produce worse results. dpc_01234: LLMs are "fundamentally... non-deterministic pattern generators" — renaming confusingly similar things in a codebase fixes performance.

**Skeptic-conversion workflow**: kevin42 describes converting skeptics with 15-30 minutes of prep: document architecture, create subagents per component, use planning mode. Everyone responds positively. The thread is full of former skeptics (ryandrake) who changed their mind with Opus 4.5 specifically.

**Programmer vs. engineer**: parliament32 gatekeeps "engineering" for novel work, drawing on Goedecke's pure/impure engineering distinction. Counterpoint: most software engineering applies known tools to non-groundbreaking problems (emodendroket).

**Bespoke software thesis**: ryandrake argues the 10% usage adage — people only use 10% of bloated software features — can now be solved by each user building their 10% for themselves. apitman: AI-built "bad code" that's smaller than bloated alternatives with tailored UX could be compelling.

The thread was partially truncated during fetch (~135 of 1,353 comments captured).
