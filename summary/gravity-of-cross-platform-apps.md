---
url: https://allenpike.com/2021/gravity-of-cross-platform-apps/
title: "The Persistent Gravity of Cross Platform"
author: Allen Pike
date_fetched: 2026-07-11
date_published: 2021-09-01
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Allen Pike (Steamclock) argues that the standard "native = better UX, cross-platform = cheaper" framing misses what actually drives platform decisions at scale. The real force is coordination cost: as product organizations grow, maintaining consistency across multiple native codebases gets quadratically harder. Cross-platform tools let teams coordinate feature work across platforms from a single codebase, and at sufficient scale that beats the UX polish native gives you.

The tradeoff, per Pike, is "coordinated featurefulness over polished simplicity." Enterprise buyers value feature checklists and 75% quality suffices for internal tools. More importantly, slow iteration kills product companies — Figma and Slack outbuilt native competitors despite not feeling fully native. Velocity wins.

The decision is never permanent. Dropbox and Slack have written about moving mobile apps *back* to fully native implementations, illustrating that the gravity shifts as team size, platform mix, and competitive context change.

A 2026 addendum argues that AI coding tools strengthen the cross-platform case: when AI can generate implementations for all platforms rapidly, the bottleneck becomes human verification of quality. One cross-platform codebase means one verification surface rather than four.
