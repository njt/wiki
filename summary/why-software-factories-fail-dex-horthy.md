---
url: https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md
title: "Why Software Factories Fail (or: harness engineering is not enough)"
author: Dex Horthy (HumanLayer)
date_fetched: 2026-08-01
date_published: 2026-07-31
---

Dex Horthy argues that "lights-off" software factories — where no human reads or writes code — fundamentally cannot work because today's coding models can't maintain codebase quality over time.

He traces the software factory from its 1968 NATO origins through the pre-AI era (humans build, humans review) into the agentic factory (agents build, humans still review) and finally the lights-off factory (agents build, automated checks replace human review). His own team tried lights-off in mid-2025 and failed repeatedly: gnarly bugs accumulated, the codebase became unmaintainable, and by November they rewrote from scratch.

The core problem is how coding models are trained. Benchmarks like SWE-bench score a binary pass/fail — did tests pass, did nothing else break. There is no penalty for eroding maintainability. The cost of bad architecture is measured in weeks or months, but RL training needs fast feedback (seconds). Since no fast, reliable oracle exists for good design, it cannot be rewarded during training. Models learn to make tests green, not to write code that stays changeable.

Claude Code's breakout success — from zero to ~$9B run-rate in under a year — came from Anthropic doing RL *inside the harness*, training the model against the exact tools it would ship with. Teams that own only the harness, not the weights, are at a structural disadvantage.

New benchmarks (SWE-Marathon, DeepSWE, Frontier Code) are starting to score maintainability, but Horthy argues they aren't there yet. Agentic code review raises the floor but doesn't move the ceiling — the ceiling is whatever RL managed to teach, and good design still can't be taught that way.

His prescription: keep humans in the loop with a four-phase planning process — Product Requirements, System Architecture, Program Design, and Vertical Slices. Program design (type signatures, call-stack trees, file-tree diffs) is the phase he finds most underemphasized. Vertical slices mean building one thin end-to-end path at a time rather than doing all database work, then all services, then all frontend. Thirty minutes of planning saves hours of review.

The pragmatic conclusion: embrace the constraints. Models are great at one-off problems and greenfield work; they're weak at maintainability. Optimize for moving 2–3x faster safely rather than chasing 10–100x and burning the codebase. For now, someone still has to read the code.
