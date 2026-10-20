# Decision Models for Evals (Hamel)

Hamel Husain's benchmark of six decision models (Jev, Perplexity Decider, Liquid d1/d1-ultra, Cloudflare Clef Flash, OpenAI Decisions) as low-cost alternatives to LLM judges for automated evals, on 98 labelled production traces from an apartment-leasing agent. His methodological thesis: these are just classifiers, so treat them like classifiers — tune the decision threshold instead of accepting the 0.50 default, set explicit precision/recall minimums, check probability calibration, and weigh cost against measured F1 on *your* data.

---

## Key quotes

> "Using this default without checking it against your data can give you subpar results."

The quiet scandal of the piece. His example — a model scoring every failure at 0.40 and every pass at 0.10 classifies perfectly at 0.30 and catches *nothing* at the default — shows that "the model was bad" is often "the threshold was lazy." A whole generation of practitioners skipped the classifiers course on the way to prompting.

> "The right tradeoff depends on your business goals. For example, criminal trials set a high bar for conviction to avoid convicting innocent people, emphasizing precision. Cancer screening may emphasize recall."

F1 is a placeholder for a decision you haven't made. Husain uses it for the comparison but insists you shouldn't — if the classifier drives automated alerting he wants a minimum of 80% on both axes, and notes the F-beta score exists precisely because precision and recall are rarely equally important. The metric choice is the product decision.

> "If you want to use scores as confidence estimates, check whether the probabilities match observed outcomes. Calibration can help, but you need to measure whether it improves them."

The most useful-and-neglected section: Clef Flash predicted ~8% failure probability for a group of traces in which 60% actually failed — a 7× miscalibration that anyone consuming scores as confidences would never notice without a reliability chart. And the honest wrinkle: Platt scaling made OpenAI's probabilities *worse* under one fitting method and better under another, so calibration itself must be evaluated, not assumed.

> "None of the ideas discussed in this post are new. Choosing a precision-recall tradeoff, tuning thresholds and comparing performance vs. cost are part of evaluating any classifier."

Both a methodological stance and a career argument — the closing line ("these are skills you already have. It's time to get revenge 🙂") tells data scientists that the AI-evals boom is their home turf. The method is old; the object (an LLM's binary verdict on an agent trace) is new.

## Key themes

#concept #tool #pattern

## Analysis

The strength of this piece is its refusal to be a leaderboard. Where vendor comparisons crown a winner, Husain's framing makes "which model" unanswerable until you've answered three prior questions: what precision/recall floor does your use case require, what threshold extracts it, and what is a false alarm worth to you? His own result undercuts any single answer — OpenAI Decisions wins on F1 but at 4.5× Perplexity's cost, and he concedes performance "varies substantially across tasks and datasets." The reproducibility discipline is unusually good for a blog post: cross-validated threshold selection, saved scores so tuning needs no new model calls, out-of-fold calibration fits, and footnotes dense enough to audit the experiment.

Two honest limitations sit in the footnotes rather than the body. First, the dataset is tiny (98 traces, five-fold CV, reliability charts of ~20 traces per bin) — every number here is indicative, not definitive, and he says so. Second, the task was deliberately broad "to make differences between models easier to see," the opposite of his own recommendation to build one judge per failure mode; he expects the gaps to shrink under realistic scoping. The piece also assumes you already have labelled traces — the human-review cost that makes this whole exercise possible is the part no decision model can classify for you.

## Related pages

This source strengthens [[The Lifecycle of LLM-as-a-Judge]]: Netflix's rubric-tuning and drift-monitoring lifecycle for LLM judges gets a cheaper cousin in decision models, plus the same calibration obsession — Husain's reliability charts are exactly the "make virtual judges trustworthy" step Airbnb describes in [[Eval-Driven Development (Airbnb)]], applied to a new model class. It nuances [[Which Decision Model Should You Use]], which compared decision models on a static benchmark; Husain's answer is that benchmark rankings are secondary to tuning thresholds and calibration on your own traces — the same model can move from useless to useful with a threshold shift. And it complicates [[LLM Judge Decision Tree]]: Turnbull routes LLM judgments into scikit-learn as features; Husain's copy-paste prompts hand the same classical-ML work (CV, Platt scaling, Brier scores) to a coding agent — the "it's a Machine Learning problem!" insight, now automated. Its calibrated-confidence gate also echoes the mechanism in [[Tiered Human-in-the-Loop at Stack Overflow]], where calibrated confidence decides what reaches a human.

---
*Sources: [[raw/decision-model-evals]], [[summary/decision-model-evals]]*
*Last updated: 2026-10-20*
