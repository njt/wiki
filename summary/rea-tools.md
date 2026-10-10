---
url: https://rea.tools/
title: "REA — Reverse engineering, with your coding agent"
author: REA (rea.tools)
date_fetched: 2026-10-10
date_published: undated
topics:
  - developer-tools
  - coding-agents-and-frameworks
---

REA is a tool that gives a coding agent the ability to inspect a running program and explain what it does — reverse engineering as an agent capability rather than a specialist's manual craft. Installation is deliberately agent-shaped: you paste a prompt into your coding agent, which runs `npx rea-agents@latest setup` and presents a setup plan for approval.

The landing page sells the idea through two worked examples. The first drives Chrome's offline dinosaur game through a local debugging connection: REA returns the actually-loaded `index.js` (with digest), the agent extracts the speed rule (`ACCELERATION: 0.001`, `MAX_SPEED: 13`, start at 6), and verifies it by calling the original update function in a controlled browser check — 4,000 updates → speed 10.0, 10,000 → 13.0. The rebuilt mini-game keeps that rule.

The second recovers Windows Calculator's `%` button semantics from native x86-64 assembly, mapping control-flow IDs (`IDC_MUL 92`, `IDC_DIV 91`) and the `/100` constant to the two behaviours users find mysterious: after `+` the percentage is taken of the previous operand (200 + 10% = 220), after `×` it is converted to a multiplier (200 × 0.1 = 20). Verification runs on a rebuilt small calculator.

Beyond the demos it advertises three scopes — native binaries (functions, strings, references, call graphs), JavaScript/Electron (modules, routes, IPC, ASAR archives), and browser/runtime activity capture with cross-run comparison — plus a starter exercise (trace CSV export in a downloadable Notes app) and provocations like cloning the site itself or reconstructing a game's gameplay logic in C from its executable.
