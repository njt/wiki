---
url: https://gist.github.com/njt/d0bc73a954e72a01d68b7077ef0da61e
title: "Harness Engineering is not Enough: Why Software Factories Fail"
author: Dex Horthy (HumanLayer)
date_fetched: 2026-07-31
date_published: 2026-07-31
topics:
  - agent-coding-workflow
---

Dex Horthy's AI Engineer talk argues that the push toward "lights-off" software factories — where agents write all code and no human reads it — is breaking codebases. Rising outages, plummeting review quality, and more bugs per developer are direct consequences, not skill issues.

The root cause is a **model training problem**, not a harness-engineering gap. Current coding benchmarks like SWE‑bench reward only test-passing correctness, not maintainability. Models learn to hack tests green with sloppy patterns (try‑catch everything, pointless casts) because the cost function of bad architecture is measured in months and years — far outside the RL reward horizon. No amount of prompt engineering, adversarial review bots, or "loops maxing" can fix this; if a model truly understood good design, it would write it in the first place.

The proposed path forward: **keep humans reading every line of code**, but shift leverage upstream. Spend 30 minutes on AI-assisted, structured pre‑planning — product review, architecture, program design, and vertical slices — to align on intent before coding. This makes review lightweight verification rather than painful rework. The talk is explicitly about complex brownfield codebases where maintainability matters, not vibe coding for side projects.

Horthy's company, **HumanLayer**, is an AI IDE and collaboration platform ("Figma for Claude Code and Cursor") that walks teams through this planning‑heavy workflow and is building "better verifiers for software quality."

The talk leaves several questions open: what concrete training methodology or verifier design would optimize for maintainability, how this workflow scales to large organizations, and whether scaling compute might eventually solve the problem despite current shortcomings.
