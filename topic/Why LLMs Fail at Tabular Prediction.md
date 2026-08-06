# Why LLMs Fail at Tabular Prediction

A rigorous empirical dissection of why frontier LLMs — so capable at text, code, and reasoning — keep losing to fifty-year-old baselines on tabular data. Garnelo and Czarnecki test five hypotheses, falsify four, and isolate dimensionality as the decisive limiting factor, while leaving the internal mechanism as an open question.

---

## Key Quotes

> "The LLM is the only method among nine whose accuracy decreases as dimensionality grows, while every classical baseline stays flat or improves."

The paper's headline result. In a random-projection sweep (which ~preserves information while varying column count), kNN, random forests, logistic regression, AdaBoost, gradient boosting, and Gaussian processes all stay flat or improve with more dimensions. The LLM alone degrades — and by the native dimensionality of most datasets, its normalised score sits at or below majority-class guessing.

> "We do not claim to have identified the internal mechanism; our results show, more modestly, that the LLM's capability dissolves with dimension in a way no noise-corrupted classical learner mimics."

An unusual degree of intellectual honesty. The paper explicitly brackets the mechanistic question — they have no access to weights or activations — and present their contribution as behavioural characterisation, not circuitry. This leaves a clean target for mechanistic interpretability work.

> "A memorisation probe first separated capability from contamination."

Before any accuracy can be read as in-context learning, you must rule out the boring explanation: the model memorised these datasets from pre-training. The probe is elegant — hold out one class entirely, train on the rest, and score against the original labels. Any genuine in-context learner must score at chance. The LLM recovers breast cancer and iris labels almost perfectly. This probe should be standard practice in any LLM-for-tabular evaluation.

> "In 6 of 20 tasks, both the predictions and the explanation qualitatively match the data-generating structure. In 4 further tasks, the predictions largely match the data, but the explanation does not. In the remaining 10 of 20 tasks, neither the predictions nor the explanation match the data."

The reasoning-trace analysis is bleak. When the model is asked to explain its predictions, explanations fail to match the data in 14 of 20 tasks. Sometimes the model predicts correctly but gives a wrong rule; sometimes both are wrong. This is consistent with known faithfulness gaps ([[The Reasoning Trap]]) and should discourage taking LLM self-explanations at face value.

> "In two dimensions the LLM predicts like a local, distance-based method (up to 91.6% grid agreement), but in higher dimensions no classical model reproduces its predictions."

Where the model works (2D), it works like a Gaussian process or 1-NN. Where it fails (higher dimensions), we can't even say *how* it's failing — no standard learner, even with tuned dimension-dependent noise, reproduces its prediction pattern. The failure is structured, not random, but opaque.

> "Anything layered on top — agentic loops, scaffolds, feature-engineering harnesses — measures the harness. We want to measure the model."

The methodological commitment that makes the paper work. By stripping away every scaffold and probing the model in pure inference mode, they isolate what the core transformer can and cannot do.

---

## Key Themes

**#concept Dimensionality collapse**: The central finding. LLM in-context classification capability dissolves as feature columns increase, even when information content is held fixed by random projection. This is specific to the LLM — no classical baseline shows this pattern.

**#pattern Hypothesis falsification by controlled intervention**: Rather than comparing benchmarks or tuning prompts, the paper constructs targeted experiments that isolate each candidate factor. H1 (separability) is tested by systematically translating class centroids. H2 (format) is tested by making the answer trivially available in a column. H3 (precision) is tested by sweeping decimal places. H4 (test load) is tested by chunking target sets. Each is cleanly rejected.

**#tool Memorisation probe**: A lightweight protocol for detecting dataset contamination that should become standard practice. The insight: hold out one class, and any accuracy against original labels can only come from prior exposure. Simple, conclusive, and currently underused.

**#pattern Distance-based behaviour in low dimensions**: In 2D, the LLM's decision boundaries are essentially those of a Gaussian process with a short length scale or 1-NN. This is a behavioural characterisation, not a mechanistic one — but it's a concrete baseline for future work.

**#concept Explanation-prediction decoupling**: The model's verbal explanations of its own predictions are unreliable — they match the data in only 6 of 20 tasks. Combined with [[The Reasoning Trap]]'s finding that reasoning amplifies hallucination, this forms a consistent picture: LLM self-explanation is diagnostically useful but should never be treated as ground truth.

**#concept Pure inference mode**: The methodological stance of testing the core model without scaffolding. This is a stronger target than it appears — it's exactly how in-context learning is advertised, and exactly the regime in which tabular foundation models operate. If the core model fails here, no scaffold can rescue it.

---

## Critical Analysis

**This paper is important because it supplies the missing causal account behind an entire research field.** The tabular foundation model literature ([[TabFM (Tabular Foundation Model)]], TabPFN, Nexus) is premised on generic LLMs being unsuited to tabular prediction. Until now, that premise was asserted, not explained. Garnelo and Czarnecki show *why* — and the answer is specific (dimensionality), not vague ("LLMs are bad at tables").

**The falsification structure is the paper's strongest methodological contribution.** Rather than the usual approach of trying to *improve* LLM performance on tables (better prompts! better serialisation! fine-tuning!), they systematically eliminate candidate explanations. This is how science is supposed to work — and it's rare in ML, where the default move is to propose a new method that beats a benchmark.

**The behavioural characterisation is both a strength and a frustration.** Identifying that 2D LLM predictions match Gaussian processes is useful. Failing to identify any classical surrogate in higher dimensions is honest but unsatisfying. The paper essentially says "the model fails in a way we can't characterise" — which is a result, but a negative one. The open question ("what IS the mechanism?") is genuine, not rhetorical.

**The memorisation probe deserves to be standardised.** It's simple, conclusive, and currently absent from most LLM-for-tabular evaluations. Any paper claiming in-context learning on classical benchmarks without running something like this probe should be treated with suspicion.

**The cost analysis is sobering and underappreciated.** $941 in API costs for a single experimental run on toy-scale datasets. A 100K-row, 20-column real-world dataset would require ~3.2M input tokens per prompt — exceeding current context windows before labels, instructions, or retries. Direct LLM tabular prediction isn't just inaccurate; it's economically infeasible at scale.

**The paper's modesty is a feature, not a bug.** "We do not claim to have identified the internal mechanism" is the kind of sentence that gets cut from most papers. Keeping it in makes the contribution clearer, not weaker: here's what we proved, here's what remains open, here's how you'd test it. This is model scientific communication.

**What I'd want next:** (1) Does the dimensionality collapse appear in other model families (the Qwen 2D replication is a start but only covers the low-dimensional case)? (2) Is the collapse monotonic with dimension, or is there a phase transition? (3) Can mechanistic interpretability tools ([[Jacobian Lens]], sparse autoencoders) identify *what* the model is doing in high dimensions that produces this pattern? (4) Does the collapse generalise to regression tasks, not just classification?

**The practitioner's takeaway is clear and negative:** stop trying to prompt-engineer your way out of this. No amount of format tweaking, precision adjustment, or batching strategy will make generic LLMs work for high-dimensional tabular prediction. Use purpose-built tabular models. The paper provides the evidence; the field now has the explanation it was missing.

---

*Sources: [[raw/2608-02412v1]], [[summary/2608-02412v1]]*
*Last updated: 2026-08-06*
