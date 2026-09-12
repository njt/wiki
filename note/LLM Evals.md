# LLM Evals

Hamel Husain's comprehensive guide to LLM evaluation, distilled from an AI evals course. The central argument: evals are not a QA step bolted on at the end -- they're the primary development activity, consuming 60-80% of your time if you're doing it right. Error analysis (manually reviewing traces to identify failure patterns) is the most important thing you can do, and most teams skip it.

---

## Key Quotes

> "Error analysis is the most important activity in evals."

> "All you get from using these prefab evals is you don't know what they actually do."

> A 70% eval pass rate often indicates more meaningful testing than 100% pass rates.

## Key Themes

#evaluation #error-analysis #testing #llm-ops #methodology

The strongest claims here: use binary pass/fail (not Likert scales), build custom annotation tools (10x faster iteration), have a single domain expert drive quality standards ("benevolent dictator"), and never outsource core error analysis. These are opinionated positions backed by course experience, and they cut against the industry trend of buying off-the-shelf eval frameworks.

The domain-specific guidance is practical: separate retrieval from generation evaluation in RAG systems, use end-to-end success metrics before diagnosing step-level failures in agentic workflows, and focus on first upstream failures in multi-turn conversations since downstream errors cascade.

Connects directly to [[Benchmark Exploitation]] -- if your evals are gameable, your development loop optimizes for gaming rather than quality. Also relevant to [[Elements of Agentic Systems Design]] which lists Evaluation as one of the ten core elements.

Hamel's "build custom evals from real failures" advice gets a structural justification from [[Goodhart's Law and AI Benchmarks]]: contamination is now the default state of any public benchmark in the web-scraping era, and the GSM1k correlation data (r² = 0.36 between memorization and score gaps) confirms that public scores measure memorization more than capability. The 20-50 task minimum viable setup Hamel recommends is also exactly the scale Alex Williams advocates for private, task-specific evaluation — convergent advice from different premises.

## Critical Analysis

Strong: the emphasis on manual error analysis over automated metrics is correct and underappreciated. The "benevolent dictator" pattern for quality standards is pragmatically wise -- consensus-driven evaluation produces mush. The minimum viable setup (20-50 manual reviews) is refreshingly low-ceremony.

Missing: almost no discussion of evaluating safety or alignment properties, which require different methodologies than task completion. The guide assumes you're building a product and measuring whether it works, not whether it's safe. For the safety angle, see [[Emotion concepts and their function in a large language model]] on monitoring internal model states.

The advice to avoid eval-driven development (writing evaluators before implementation) is interesting and contrarian -- it implies you need to see real failures before you can write meaningful tests, which is the opposite of TDD orthodoxy.

Airbnb's [[Eval-Driven Development (Airbnb)]] embraces the EDD label that Hamel explicitly rejects, but the disagreement may be semantic: both agree you shouldn't write evals in a vacuum. Airbnb's version is "let real errors guide your metrics" and "build your prototype, run it through 100 examples, then read the outputs" — nearly identical to Hamel's "manually review traces to identify failure patterns." Where Hamel says "don't write evaluators before implementation," Airbnb says "build infrastructure to discover, encode, and continuously test for failure modes as they appear." Both converge on the same practice from opposite framings. Airbnb also adds what Hamel's guide lacks: a detailed calibration methodology (golden dataset, Cohen's kappa, iterative rubric refinement) for making LLM-as-judge evaluators trustworthy.

---
*Sources: [[summary/llm-evals]]*
*Last updated: 2026-08-07*
