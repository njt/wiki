# Domain-Specific Hallucination Detection in Large Language Models

This arXiv paper (Chundru et al., Sep 2026) builds a response-level hallucination detector from three stacked signals — a fine-tuned DeBERTa-v3 classifier, MC Dropout uncertainty, and temperature-scaled calibration — achieving F1=0.915 on HaluEval, then uses that detector to drive DPO training that cuts a small Qwen generator's hallucination rate by more than half. Its punchline for practitioners: general-domain detectors transfer poorly to specialist domains, and domain-matched pre-training beats everything else.

---

## Key quotes

> "Cross-domain evaluation on the SciFact biomedical benchmark shows that general-domain training transfers poorly (F1=0.52), motivating domain-specific fine-tuning. PubMedBERT fine-tuned on SciFact achieves F1=0.63 and AUROC=0.81, demonstrating that domain-matched pre-training is the strongest adaptation strategy."

The most transferable finding in the paper. A detector that looks great on HaluEval (0.915) drops to near-coin-flip (0.52) on biomedical text — a warning to anyone importing a general hallucination scorer as a production [[Ways of Checking|gate]]. The fix is unglamorous: fine-tune on your own domain.

> "A context ablation study confirms the model performs genuine entailment reasoning rather than exploiting surface patterns, with summarization F1 dropping 24% when knowledge context is removed."

A rare, welcome piece of mechanistic hygiene. Most detector papers skip this check; without it, you can't distinguish a detector from a benchmark-overfitting shortcut learner (the same failure mode [[TrapQA — Testing Reasoning Against Priors|TrapQA]] probes from the other direction).

> "we apply Direct Preference Optimization (DPO) to a Qwen2.5-0.5B generator, reducing its hallucination rate from 85.5% to 37.7% (55.9% relative reduction) as measured by our detector."

Detector-as-reward-model: the classifier isn't just an evaluation artifact, it becomes the training signal. That closes a loop — but note the metric is circular: the generator is being optimised against its own judge, so detector errors become blind spots in the fix. A 0.915 F1 judge is good, not perfect.

> "Learning curve analysis reveals that 25% of training data captures 77% of full-data performance."

Data efficiency matters more than headline numbers for anyone actually building this: the last 75% of labelling buys only ~23 points of performance.

## Themes

- #concept — hallucination detection as a stacked multi-signal classifier problem, not a prompt trick
- #concept — detector-as-training-signal (DPO against your own judge)
- #pattern — domain-matched fine-tuning beats general transfer; labels have diminishing returns
- #tool — released code and models make this directly reusable as a verification gate

## Opinionated take

This is a solid, unglamorous systems paper: no new architecture, just three known signals combined carefully and ablated honestly. The 93.2% accuracy with MC Dropout is real but paid for at inference time — MC Dropout means multiple stochastic forward passes, so anyone bolting this onto an agent loop should expect a latency multiplier. The genuinely interesting move is the detector/DPO coupling: it is a miniature version of the self-correcting loop that [[Guardrails and Feedback Loops]] pages describe, built at paper scale. The caveat that goes unmentioned loudly enough: self-measured hallucination reduction is exactly the kind of number [[Goodhart's Law Comes for Every Benchmark You Trust]] should make you suspicious of. And the domain-transfer result quietly indicts the industry habit of shipping general-purpose judge models into specialist pipelines.

## Related pages

- Strengthens [[Ways of Checking]] with a concrete, measured instance of a verification gate — and a hard number for how badly such gates degrade outside their training domain.
- Complicates [[TrapQA — Testing Reasoning Against Priors]] from the other side: that source tests whether reasoners beat priors; this one shows a detector must prove it reasons (not pattern-matches), and supplies the ablation protocol to do it.
- Nuances [[LLM Evals]]: detectors trained on benchmarks inherit benchmark fragility, and the HaluEval→SciFact cliff is a case study in why single-benchmark scores mislead.
- Feeds the [[Guardrails and Feedback Loops]] theme that quality must be self-correcting — here literally, with the detector's output used as the DPO reward signal.

---
*Sources: [[raw/2609-11878]], [[summary/2609-11878]]*
*Last updated: 2026-10-11*
