---
url: https://www.oreilly.com/radar/so-long-and-thanks-for-all-the-context/
title: "So Long and Thanks for All the Context"
author: Andrew Stellman
date_fetched: 2026-07-18
date_published: 2026-06-25
topics:
  - agent-memory-and-context
---

Stellman explores the "U-shape" failure — the well-documented tendency of LLMs to attend strongly to the beginning and end of context while ignoring the middle. The problem is structural: a 2026 paper by Borun Chowdhury at Meta proved it exists at model initialization, before any training. Larger context windows make single-fact retrieval better but make the middle larger and more dangerous for sustained agent work.

The article offers five techniques for managing the problem. **Curate, don't accumulate** — clear context and reload only what matters, using a self-contained "context brief" document. **Position critical information at the edges** — put load-bearing instructions at the start and end of the prompt. **Run many short sessions instead of one long one** — treat the agent as a pipe, not a database, with state written to disk and fresh restarts built into the process. **Restate key info close to the point of use** — force the agent to re-read source files at the moment of writing rather than paraphrasing from memory. **Test the middle** — cross-check what the agent claims against what's on disk to catch U-shape failures before they compound.

The core discipline: keep the working set small, put load-bearing information at the edges, and verify claims against ground truth. The author frames this as a working-memory problem that echoes computing history — information an agent depends on must live somewhere more durable than working memory.
