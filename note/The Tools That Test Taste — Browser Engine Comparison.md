# The Tools That Test Taste — Browser Engine Comparison

A pilot study from uipalooza.ai comparing Chrome, Obscura, and SleepyHollow as the rendering engine behind an automated evaluator of saved AI-generated designs. The lesson generalises far past this experiment: when an agent judges output through a tool, the tool's fidelity silently becomes part of the evaluation, and a capture that "returns success" is not the same as an image that is correct or an interaction that actually happened.

---

72 primary images (4 briefs × 3 designs × 2 viewports × 3 engines) plus 24 repeatability captures, rendered with matched viewports, a common readiness probe, and a shared proxy that disables CSS animation. Everything — screenshots, hashes, measurements, review notes — saved locally for audit.

## Key Quotes

> "Capture success means a command returned successfully and produced a PNG, not that the image is correct."

The single most quotable line, and the reason this report belongs in a wiki about agent evaluation. Any screenshot-based judge inherits this failure mode: the pipeline's success signal and the judge's evidentiary basis are two different things. Compare the framing in [[Evaluating AI-Generated Frontend Code — What Should We Actually Test]] — visual evidence is precisely the evidence least correlated with whether the UI works; this source adds that even the visual evidence itself can be corrupted by the rendering layer.

> "Obscura's covered-button click returns success without activating the button; its Enter key also fails to activate our focused button."

Silent false positives in interaction tests are worse than rendering diffs, because they're invisible in the artifact. An evaluator that logs "click succeeded" on a click that did nothing is manufacturing proof of quality. This is the harness-design version of a classic eval trap: the tool's telemetry is not the ground truth.

> "The alternative captures save roughly a third of a second at the median here. That does not resolve a model spending 30 minutes debugging browsers. A controlled preview tool and reliable lifecycle matter more."

A refreshing cost framing — correctness and reliability of the eval infrastructure dominate micro-latency, and the thing that actually costs time is an agent debugging flaky tooling. Compare the reliability-over-speed argument in [[Your Agent Should Run Its Own Observability Stack]].

> "Neither should silently replace the current evaluator."

The conservative recommendation is itself a finding: swapping the rendering backend of an evaluator is not a drop-in infrastructure change, it's a re-baselining of the benchmark.

## Themes

- #concept — capture success ≠ capture correctness; tool fidelity is part of eval fidelity
- #pattern — controlled eval corpora: matched viewports, readiness probes, animation-off proxies, saved hashes for auditability
- #tool — Chrome, Obscura, SleepyHollow as rendering backends for automated design judging

## Analysis

The most interesting thing here is what it says about evaluation stacks built by and for agents. The evaluator in question judges AI-generated designs through screenshots; this report is a meta-evaluation of the eyeball of that judge. Obscura — which positions itself elsewhere in this wiki as agent-first browsing infrastructure — fails precisely because its rendering pipeline isn't Chrome-faithful: dropped text, changed crops, missing map labels. For scraping, that's an acceptable trade; for visual judgment, it's disqualifying. It's a sharp reminder that "agent-grade browser" is not one product category.

The interaction findings matter even more. A click that reports success without activating the button poisons any downstream agent loop that treats tool telemetry as evidence. The methodological discipline (fixed corpus, hashes, honest caveats about what's *not* tested — no full-page, no touch, no memory ranking) is a good model for how to write up a tool comparison that's meant to feed a real decision.

#concept #tool #pattern

---
*Sources: [[raw/browser-comparison]], [[summary/browser-comparison]]*
*Last updated: 2026-10-11*
