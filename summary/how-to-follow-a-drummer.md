---
url: https://drummate.app/blog/how-to-follow-a-drummer
title: "How to Follow a Drummer"
author: Sashyo
date_fetched: 2026-07-11
date_published: 2026-07
topics:
  - developer-tools
---

A blog post from the creator of DrumMate explaining the engineering problems behind
building a system where the drummer leads and the machine follows — the inverse of
almost all electronic music setups.

The core insight is that drum hits are *evidence*, not commands. Treating a kick
pattern as a clock fails because syncopation, fills, and rests produce wildly uneven
intervals. Instead, the author built a software phase-locked loop: a free-running
clock steered by a beat-tracker that fits incoming hits against a grid hypothesis
(period + phase), nudging on agreement and mostly ignoring hits that disagree.

Key design decisions: clock division is causal and robust; clock multiplication
requires prediction and is where the hard problem lives. The clock must bend, not
jump — sensitivity is tuned so small deviations read as feel (ignored) while
sustained drift reads as a deliberate tempo change (followed). The system schedules
notes ahead against a forecast rather than reacting to each hit, keeping audio
latency out of the drummer's timing loop.

The most instructive failure: early versions lost lock whenever the drummer got
interesting, which was fixed by *coasting* — holding the last good estimate through
confidence dips rather than stopping. The post also references James Holden's mutual
entrainment work and notes that a human tapping along still beats any algorithmic
tempo tracker, since brains predict rather than react.

The ideas were implemented in DrumMate (Android app) but apply across platforms — Eurorack modules, Max patches, plugins.
