---
url: https://sunflowersgame.com/
title: "Sunflowers — Difficulty Tuning"
author: Sunflowers game (sunflowersgame.com)
date_fetched: 2026-10-11
date_published: unknown
topics:
  - misc
  - software-engineering-craft
---

Sunflowers is a flower-planting logic puzzle game whose site doubles as the documentation for its difficulty system. Every generated grid is solved end-to-end by the same logic engine that powers the in-game hints, which always plays the *simplest* move available, exactly like a careful human, and records which reasoning techniques the grid forced it to use.

There are five techniques, and each difficulty tier is named for the hardest one the grid demands. Basic settles squares with a single clue (Easy). Subset subtracts one clue from another (Medium). Overlap uses min/max bounds on a shared region (Hard). What-if plants a hypothesis and follows the rules until a clue breaks — cousin of Sudoku's forcing chains, capped at a few counting steps (Extreme). Beyond the cap sits Insane, which demands longer chains plus the either-way technique, a cousin of Sudoku colouring where a pivot is tried both ways and both chains agree on another square with no contradiction at all.

Difficulty is made falsifiable: Medium grids must force a minimum count of subset moves, Hard a minimum of overlap moves, Extreme a minimum of what-if moves with at least one chain of given depth. A 0–100 score places the grid inside non-overlapping band ranges so a 62 is always genuinely harder than a 40, and estimated solve time sums per-technique time weights that live in the source (`EST_TIME`). Even the 0 tiles follow a per-tier budget (`ZERO_CLUE_RULES`), and the generator protects, strips, or injects them to comply. Each click tries up to 90 candidates; if none fits the sliders, the closest match is served and flagged.
