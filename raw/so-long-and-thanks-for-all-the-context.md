---
url: https://www.oreilly.com/radar/so-long-and-thanks-for-all-the-context/
title: So Long and Thanks for All the Context
author: Andrew Stellman
date_fetched: 2026-07-18
date_published: 2026-06-25
---

# So Long and Thanks for All the Context

**Author:** Andrew Stellman
**Publication:** O'Reilly Radar
**Date:** June 25, 2026
**Reading time:** 18 minutes
**Topics:** AI & ML, Business, Data, Innovation, Research, Security
**Series:** Fourth article in a "context management trilogy" (the fourth installment of a self-described trilogy)

## Opening

The piece opens with editor Mike Loukides asking about "the tendency for a model to ignore the middle of the context," particularly in models with very large context windows. The author confirms this is an excellent question and that clearing and reloading context is a "stopgap" solution.

## The U-Shape Failure

Stellman describes encountering this failure mode while running his open-source Quality Playbook — a code quality engineering skill. During a bug writeup phase, the agent produced "skeletal-looking stub files" with blank template values instead of populated details, despite having all necessary information in its context window.

The U-shape is described as an active area of academic research. The author references Nelson Liu's "Lost in the Middle" paper (arXiv:2307.03172), which demonstrated that models perform best when relevant information sits at the beginning or end of context, and worst when it's in the middle. The field uses the terms "primacy bias" and "recency bias" for these preferences.

The article cites a 2025 ICML paper arguing the U-shape is a structural property of transformer architecture — an "equilibrium between two opposing forces" — and a 2026 paper by Borun Chowdhury at Meta proving mathematically that "the U-shape exists at the moment of initialization, before any training has happened."

## On Large Context Windows

The author notes that while models like Gemini have achieved near-perfect single-needle recall at 1M tokens, "a two-million-token window means a bigger middle to fall into." The key insight: "Bigger windows have made simple single-fact retrieval much better. They have not made long-context agent work reliable."

## Five Techniques for Managing U-Shaped Context Problems

### 1. Curate, Don't Accumulate

The brute-force version of what Mike suggested: clear context and reload with only what matters. During v1.5.2 release prep, Stellman worked with the AI to write a context brief — a separate document containing everything the implementing session needed. He started fresh sessions against the brief rather than continuing long-running ones. Three reasons emerged: the brief was self-contained, fresh context enforces stricter adherence to instructions, and the brief becomes a reproducible audit trail.

### 2. Position Critical Information at the Edges

Put load-bearing information at the beginning and end of context. Using tools like Claude Code's `--append-system-prompt` places important info where the model attends to it most. The middle should hold less important material.

### 3. Short Sessions Over Long Ones

"Don't run one long session. Run many short ones, each reading fresh from disk." The author describes building a system to index chat history from multiple AI tools, using Haiku 4.5 to summarize interactions. The first attempt failed because the model tried to keep state in its head. The fix was reframing the architecture: "it doesn't need to remember things, it just needs to write them down as they go." The protocol included resuming from a cursor in `progress.json`, updating after every line, and expecting to run out of context before finishing — "build fresh restarts into the process."

### 4. Restate Key Info Close to the Point of Use

This technique fixed the Quality Playbook stub-file problem. The fix was a prompt instructing the agent to re-read the source file before writing, with the critical line: "Do not paraphrase from memory." This forced "a fresh read of the file at the moment of writing."

### 5. Test the Middle

Catch U-shape failures by running deterministic checks. The pattern: "compare what the agent claims to know against what's on disk." The Haiku summarizer's resume protocol cross-checked `progress.json` against the actual last line written to the summary file. When they disagree, "the agent flags the discrepancy and stops before adding any new work on top of a broken state."

## The Core Discipline

The author concludes that context loss and lost-in-the-middle are both "problems of working-memory unreliability," and the discipline is the same: "keep the working set small, put the load-bearing information at the edges of the window, and check the agent's claims against ground truth on disk."

The closing reflection ties back to computing history — from 32KB core memory in the 1970s to the author's 286 in the 1980s to modern context windows: "if your agent's ability to do its job depends on information, that information needs to live somewhere more durable than working memory."

## Key Concepts

- **U-shape**: The finding that models attend best to the start and end of context, worst to the middle
- **Primacy bias**: Preference for information at the beginning of context
- **Recency bias**: Preference for information at the end of context
- **Context brief**: A separate document capturing everything a fresh session needs to know
- **Externalize-recognize-rehydrate**: Pattern from the prior article for detecting and recovering from context loss
- **Short sessions discipline**: Treating the agent like a pipe, not a database — state lives on disk
