---
url: https://danluu.com/astra-testing/
title: "Testing techniques with GPT-6 Astra"
author: Dan Luu
date_fetched: 2026-11-20
date_published: 2026-11-19
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

Dan Luu re-runs his Zstd test/verification-technique eval with GPT-6 Astra and finds the 5.6 Sol picture overturned: capability is now high enough that the Zstd benchmark is saturated (99%+ holdout correctness for ~$1, versus 63–77% at $3.50–$5.50 for 5.6 Sol), so he builds a harder Pandoc eval (~$90k of MSRP tokens, 25 runs per condition, ~14 turns average) where no condition reaches 100% correctness.

Unlike the earlier eval — where almost no prescribed technique beat plain Default prompting and the "tests" agents wrote were a caricature of each technique — many conditions now beat Default on Pandoc, though wide conceptual error bars mean we can't say which ones truly do. Interesting signals: differential testing drove genuine examination of boundary conditions; TDD underperformed again; ACL2 performed well (again); and property-based testing finally produced tests that behave as PBT should.

Luu spends much of the post on epistemic honesty: small samples, path dependence, prompt-sensitivity, and one-Pandoc-sized-problem all limit generalization; he argues you'd need ~100x the runs to say anything definitive. He also pushes back on the viral "tell your agents to stop writing tests" claim built on DeepSWE, and — in appendices — asks why AI labs don't publish technique comparisons, includes David MacIver's observation that Claude is far more proactive about PBT than ChatGPT, and reports Roman Leventov's negative case study where generated tests found only 10% of bugs at the cost of a 300k-line harness.
