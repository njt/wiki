---
url: https://github.com/jessewaites/antiquity
title: "Antiquity"
author: Jesse Waites
date_fetched: 2026-10-10
date_published: 2026 (repository)
topics:
  - coding-agents-and-frameworks
  - agent-coding-workflow
---

Antiquity is a small Go CLI (`antiquity new my-investigation`) that scaffolds an opinionated workspace for AI-assisted historical investigation: a folder tree, an `AGENTS.md` rulebook, templates, a `keys.yml` budget file, and a git pre-commit hook that blocks API keys. It does not run any pipeline itself — your coding agent does the work; Antiquity supplies the conventions and the traps learned from a real investigation that used an AI pipeline over digitised colonial archives (and found a lost meteorite fall).

The generated `AGENTS.md` encodes a seven-step funnel — question plus known-answer control, wide retrieval (regex + embeddings), a cheap judge model over all candidates, an expensive reader model over survivors, verification on the original page image, novelty check against catalogues, then evidence and write-up — along with hard rules ("nothing is a find before the page image", "every negative result needs a control", budget caps per run). The tool itself is ~1,700 lines of Go using Bubble Tea v2: an animated title screen, a four-question wizard (skippable with ctrl+s for agent use via `--yes`), and a template-rendering scaffolder.

The interesting artifact is the codified process, not the code: judge/reader/vision model roles configured provider-neutrally in YAML, funnel-stage counts and cost logged per run, candidate/evidence/reject folders with naming conventions, and a `lessons.md` that feeds word-traps back into queries and judge questions.
