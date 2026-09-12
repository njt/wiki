---
url: https://noelo.org/blog/kuna-release/
title: "Kuna: Decompiler Development in the Age of Coding Agents"
author: Zion Leonahenahe Basque
date_fetched: 2026-08-01
date_published: 2026-07-29
topics:
  - agent-coding-workflow
---

Kuna is an experimental "agent-first decompiler designed for autonomous
refinement," announced by Zion Leonahenahe Basque (UGA / AFRL / Metalware). Its
central claim: an LLM wrote nearly every line of Kuna, yet it rivals IDA Pro
9.2 in control-flow structuring on C programs.

On decbench.com benchmarks, Kuna achieves perfect structuring on 44.4% of
functions versus IDA's 45.7%. This was reached through autonomous refinement —
the LLM studies cases where it underperforms IDA and iteratively improves. The
same process reimplemented more than 20 fundamental features from the angr
decompiler that took years of human research.

Basque names three limitations. First, Kuna is possible only because of decades
of prior decompiler research; technically it's a Rust port of Ghidra reworked to
match angr's pipeline, and angr remains the home for frontier algorithms.
Second, autonomous refinement still depends on human scientific insight —
choosing metrics, benchmarks, and what constitutes meaningful improvement.
Third, the work so far covers only control-flow structuring; types,
optimizations, recompilability, and variable identification remain open
questions for the framework.
