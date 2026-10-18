---
url: https://github.com/echohack/factory
title: "factory — A skill for building software architecture diagrams as a Factorio factory"
author: echohack
date_fetched: 2026-10-18
date_published: 2026
topics:
  - claude-code
  - software-engineering-craft
---

Factory is a Claude Code Agent Skill that turns architecture explanations into **working, animated Factorio factories**: belts carry values, inserters do handoffs, assembler machines do work, chests hold state. Invoked as `/factory explain <topic>`, it reads your code or planning docs, writes an architecture inventory, then builds a browser-rendered simulation from a library of 36 factory patterns (worker rows, splitter fan-outs, main buses, backpressure) drawn with genuine Factorio 2.0 sprite art imported from the user's own copy of the game.

The repo is a standard Agent Skill — a `SKILL.md` plus reusable code — split into `kit/` (scene renderer, deterministic trace player, sprite atlas), `catalog/` (68 world objects and 28 item tokens, each with a fixed vocabulary role: iron plate is always a record, copper cable a packet), `patterns/` (six families of arrangements with machine-checkable ground rules enforced by `checkLayout`), `examples/` (a 275-line OBS Studio pipeline model including scenarios for network congestion and dropped P-frames), and `scripts/` for asset import and verification. It runs through the Vite+ CLI, serves figures at `factory.localhost`, and tests enforce both layout legality and conservation of values.

What makes it notable as a skill design: rather than hoping the model draws well, it gives the agent a shared design language (fixed pattern meanings, fixed token semantics), a legal-geometry checker, and a verification checklist — so the output is constrained by the same physics players already know from the game. The README pitches it as a replacement for mermaid and UML in planning work.
