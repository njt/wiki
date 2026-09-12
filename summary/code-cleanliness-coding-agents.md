---
url: https://arxiv.org/html/2605.20049v1
title: "Does Code Cleanliness Affect Coding Agents? A Controlled Minimal-Pair Study"
author: Priyansh Trivedi, Olivier Schmitt (SonarSource)
date_fetched: 2026-07-08
date_published: 2026-05
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

This paper from SonarSource asks whether code cleanliness (measured by static-analysis violations and cognitive complexity) changes how well coding agents can navigate and modify a codebase. The key methodological contribution is **minimal-pair repositories**: behaviorally equivalent codebases that differ only on cleanliness, created either by degrading a clean codebase (Slopify) or cleaning up a messy one (Vibeclean).

Across 660 trials (33 tasks × 10 runs × 2 sides) using Claude Code with Claude Sonnet 4.6, cleanliness did **not** change pass rates — both cleaner and messier sides landed around 91–92%. But it did change the agent's operational footprint: cleaner code used **7–8% fewer tokens** and triggered **34% fewer file revisitations**. The authors interpret this as the agent reading wider on its first pass through clean code, versus repeatedly returning to verify edits in messier code.

The effect is strongest on **multi-module tasks**, where input tokens dropped 10.7% and revisitations halved. On cognitive-hotspot tasks (dense single-method complexity), the token advantage mostly vanished — cleanup pipelines often spread complexity across more methods rather than eliminating it. Two case studies illustrate the nuance: when cleanup replaced god-method dispatchers with named helpers, agents could grep precisely and saved 35% on tokens; when cleanup added structure around focal logic without shrinking the file, agents paid for the added surface area.

The authors also control for comment volume (cleaner variants had more docstrings and suppression markers) and find it does not explain the effect. Key limitations: only one agent/model configuration was tested, tasks were short-horizon, and the authors curated everything end-to-end. The central claim is that traditional maintainability principles remain relevant in the era of AI-driven development — cleaner code helps agents work more efficiently, even when it doesn't change whether they succeed.
