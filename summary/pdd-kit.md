---
url: https://github.com/itai-gendler/pdd-kit
title: "pdd-kit — Product-Driven Development for coding agents"
author: Itai Gendler
date_fetched: 2026-10-20
date_published: 2026 (ongoing, v0.37.0)
topics:
  - specifications-as-the-product
  - coding-agents-and-frameworks
---

pdd-kit is an open-source (MIT) npm toolkit (`pdd-toolkit`) that implements "Product-Driven Development": instead of writing a spec per change like spec-driven development tools (OpenSpec, spec-kit, Kiro), it maintains a single git-tracked **Product Model** in a `product-model/` folder of plain Markdown files, describing what the product *is* — Entities, Interfaces, Domains, Capabilities, Journeys, Business Rules and Variations — apart from how it's built.

The workflow runs through three agent skills installed into Claude Code, Codex, Cursor, Gemini CLI or Copilot: `/pdd-map` reverse-engineers an existing repo into a model by reading code without running it; `/pdd-ideate` records intended product changes in the model before any code is written; `/pdd-verify` checks code against the model, classifying each gap (aligned, model-right, code-right, neither-right) and fixing only what the user approves. A CLI (`install`, `lint`, `view`, `blueprint pull`) lints the model structurally and opens it as a local browser report; Blueprints are pre-reviewed models (e.g. a kanban board, a team wiki) to start from instead of a blank page.

The project's argument: agents write code faster than anyone can explain the product to them, so they guess, and the guesses ship. One shared, reviewable product definition — reviewed in the same PR as the code, with a verify skill that checks the code against it — turns "done" into "matches the approved model." The repo is dogfooded: pdd-kit's own product model lives in its repository.
