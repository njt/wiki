---
url: https://medium.com/airbnb-engineering/eval-driven-development-lessons-from-evaluating-genai-at-scale-e817e5ae5788
title: "Eval-driven development: Lessons from evaluating GenAI at scale"
author: Rohit Girme, Dan Miller, Mia Zhao, Lifan Yang, Clint Kelly
site: Airbnb Engineering (Medium)
date_published: 2026-07-29
date_fetched: 2026-08-07
---

Airbnb's infrastructure team shares production-hardened evaluation practices for LLM-powered features. The central argument: evaluation is not a QA afterthought — it's a first-class engineering discipline that should consume a meaningful share of project effort. Without deliberate strategy, teams fall into three traps: false confidence from generic metrics, undetected regressions from unmeasured dimensions, and wasted effort on eval pipelines uncorrelated with outcomes.

**Eval-driven development (EDD)** is the GenAI analogue of TDD: build infrastructure to discover, encode, and continuously test for failure modes as they appear. Five principles anchor it: define goals and gates upfront, let real errors guide your metrics, keep evaluators small and sharp (3–5 well-calibrated judges beat 20–30 noisy ones), appoint a human decision-maker, and collaborate continuously with product partners.

**Three evaluation layers** form the toolkit: programmatic checks (fast, catches format/JSON failures), LLM-as-judge (nuanced quality assessment via carefully designed rubrics), and human evaluation (gold standard for ground truth and edge cases). The rule of thumb: start with 20–100 rows labeled by subject-matter experts, and never automate until human disagreement is resolved.

**Calibration** is the critical step that makes virtual judges trustworthy. Create a golden dataset of 50–100 examples that must include bad ones, measure agreement (target high 80s–90s% via Cohen's kappa or Krippendorff's alpha), then iterate on the rubric and few-shot examples until the target is hit. An uncalibrated judge is worse than none — it gives false confidence.

**Agentic system evaluation** requires assessing trajectories (reasoning paths, tool calls, intermediate states) not just final outputs. A correct answer can mask a broken reasoning path. The article recommends using trace data from observability platforms and DFS traversal to verify correct subagent invocation and tool usage.

The practical walkthrough demonstrates EDD end-to-end: discover failure modes from 100 prototype runs → build targeted evals per failure dimension → calibrate judges to human agreement → scale to 5,000 examples and production monitoring with 5% daily sampling. The meta-lesson: fix one variable at a time (model, then prompt, then serving config), letting evaluators and candidates sharpen each other until both stabilize.
