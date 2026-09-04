# Analytical AI

Sutro's handbook opens by naming a category most AI coverage ignores: **analytical AI** — the usage pattern where a foundation model's job is to *decide* something rather than *create* something. Data, research, ops, and product teams use LLMs to process unstructured data and make scaled operational decisions. Sutro's contribution is to argue the distinction isn't cosmetic: because analytical tasks are measurable, discriminative, and latency-tolerant, their best practices diverge sharply from generative use — ground-truth evals, the smallest evaluated model, and batch/OLAP-style processing.

---

## Key Quotes

> "If the AI's job is to decide something, rather than create something, it's analytical AI."

The one-sentence definition, and a clean dividing line. It reframes "AI" from a single activity into at least two: generation (open-ended, creative, unmeasurable) and decision (closed, discriminative, verifiable). Most of the wiki's pages on evals, judges, and verification are describing analytical AI without naming it — this gives the category a handle.

> "Tasks are typically measurable. You can create a ground-truth dataset using expert annotations that can be validated against for correctness."

This is the hinge the whole divergence rests on. Generation has no ground truth, which is why it forces you to build evals as a proxy — and Sutro notes evals are *themselves* analytical AI. A judge deciding "is this output correct?" is a decision task, not a creative one. That's why [[LLM-as-a-Verifier]]'s continuous-scoring trick works: it treats verification as a measurement problem.

> "Reduce 'creativity' in favor of consistency... the task can often be run on the smallest possible model that's been evaluated for task accuracy, rather than reaching for the largest, maximally-intelligent model."

The economic corollary of discriminative tasks. [[Fine-Tuning a Local LLM to Categorize Questions]] is this claim in miniature — a 600M model hitting 92% on classification — and [[Model Routing Is Simple Until It Isn't]] is the production caveat: "smallest evaluated model" assumes you can predict task difficulty and cache economics at routing time.

> "Analytical AI typically does not involve a transaction with a user, [so] more latency is tolerated — batch and other flexible workload processing models are acceptable... analogous to OLTP vs. OLAP."

The least-quoted but most consequential point. Decision workloads decouple from interactive latency, which collapses the cost curve: overnight batch beats real-time serving by orders of magnitude. This is the same insight as [[The Tokens You Can't Wait For]] from the serving side — when you can wait, you can batch, and batching turns idle compute into saturated assets.

## Key Themes

- **#concept** — Analytical vs. generative AI as a real boundary, not a marketing split. The distinction cashes out in measurability, model choice, and workload shape.
- **#pattern** — Ground-truth evals as the foundation. If you can annotate a golden set, you can validate; validation is what lets you use smaller models safely.
- **#concept** — Evals are analytical AI. Judging and verifying are decision tasks, so the entire eval/verification toolchain inherits analytical AI's properties.
- **#pattern** — OLTP vs. OLAP for inference. Latency tolerance means batch processing, which changes the economics of inference entirely.
- **#tool** — Sutro, a startup building products to support analytical AI, whose handbook is an evolving FAQ from customer trenches.

## Critical Analysis

Sutro's framing is genuinely useful, and it arrives with vendor transparency: they build for this space, and the handbook is as much a positioning document as a reference. That doesn't undermine the categories — the "decide vs. create" split, the measurability claim, and the OLTP/OLAP analogy are all sharp — but the handbook overstates how clean the boundary is.

The measurability claim deserves the most scrutiny. "Analytical tasks are measurable" is true for the cases Sutro has in view (extraction, classification, structured decisions), but it quietly assumes you can afford expert annotations. [[Eval-Driven Development (Airbnb)]] shows what it actually costs: golden datasets of 50–100 examples with failures, judge calibration to high-80s agreement, and a meaningful share of project effort. Measurability is not free — it's purchased with annotation labor and calibration loops. Analytical AI doesn't escape the eval problem; it *is* the eval problem.

The "smallest evaluated model" advice also hides a trap. It's correct that discriminative tasks don't need frontier creativity, but "evaluated for task accuracy" is doing enormous work: you must have built and maintained the ground truth first. [[Local Qwen Is Not a Worse Opus]]'s "analysis, not interpretation" boundary is the same instinct from a founder running small local models — let the model surface patterns, don't let it draw conclusions — and its looping failures show what happens when the model outruns its evaluation. The Sutro handbook, as an introduction, gestures at all of this through its four-section structure (Primitives, Patterns, Architectures, Deployment) but the pages behind those headings carry the actual weight.

The strongest thread to pull on is the one Sutro states and moves past: **evals are analytical AI.** If judging and verification are decision tasks, then the entire [[Guardrails and Feedback Loops]] ecosystem — [[LLM-as-a-Verifier]], [[The Lifecycle of LLM-as-a-Judge]], [[LLM Evals]] — is one big analytical-AI workload. That single observation unifies a set of pages the wiki currently treats as separate concerns, and it's the most portable idea in the handbook.

---

*Sources: [[raw/handbook-sutro-sh]], [[summary/handbook-sutro-sh]]*
*Last updated: 2026-09-04*
