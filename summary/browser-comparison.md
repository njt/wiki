---
url: https://uipalooza.ai/experiments/browser-comparison/
title: "A test of the tools that test taste"
author: uipalooza.ai
date_fetched: 2026-10-11
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - developer-tools
---

A short, dense report comparing three browser engines — Chrome, Obscura, and SleepyHollow — as rendering backends for an automated evaluator of AI-generated designs. The corpus: 4 briefs × 3 saved designs × 2 viewports × 3 engines, 72 primary screenshots plus 24 repeatability captures, with source hashes and measurements saved locally.

The headline finding is that "capture success" is not correctness. Obscura loses door text in Sidewalk, changes image crops and wrapping, and drops map labels in Elsewhere; SleepyHollow preserves composition with only native-control and typography differences. Interaction fidelity diverges too: SleepyHollow's selector click activates a covered button and its coordinate click hits an overlay, while Obscura's covered-button click "returns success without activating the button" — a silent false positive.

Methodological care is a theme: matched viewports, 1× scale, a common readiness probe, CSS animations disabled through a shared proxy. Pixel differences include font rasterization and are explicitly not design scores; memory ranking was deliberately skipped as misleading. The recommendation: SleepyHollow merits a managed preview pilot, Obscura would distort this benchmark, and neither should silently replace the current evaluator — median capture speedups of a third of a second don't resolve a model spending 30 minutes debugging browsers.
