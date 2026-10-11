# The 70–75% Ceiling in Automated Deception Detection

A meta-analysis of 25 years of automated verbal deception detection (289 reports, 6,136 models, 3,653 meta-analyzed) finds pooled accuracy of 74.4% — statistically indistinguishable from manual, theory-driven methods, implying a hard ceiling that neither embeddings, transformers, nor LLMs have moved. Accuracy is driven by methodological quality (ground truth, data source, evaluation procedure) far more than by model complexity.

---

This is a paper about models, but its real subject is research methodology. The Tilburg/UCL team coded six levels of ground-truth quality and found only 12.46% of reports use fully verifiable labels (mock crimes, object descriptions); the modal report (38%) uses experimental assignment with no verification of what participants actually wrote, and a quarter use labels inferred from human raters — who are, per the same literature, near chance at detecting deception. The moderator analysis quantifies the damage: non-verifiable ground truth *inflates* accuracy by ~5.7 points, while indirectly inferred labels collapse it by a staggering ~69 points (a confounded estimate, but the direction is unmistakable). The field's benchmark numbers are, to a significant degree, artifacts of its labeling practices.

The computational story is the more humbling half. Embeddings over simple dictionary/statistical features: +1.5 points. Fine-tuned transformers over statistical learning: +3.5 points. LLMs by prompting: *−1.2 points* — worse than logistic regression and SVMs. A quarter century of NLP revolution, and the best move a deception classifier can make is a fine-tuned BERT. The authors also flag that the most-reused dataset (Deceptive Opinion Spam) has a built-in confound — genuine reviews scraped from TripAdvisor, fake ones crowdsourced on MTurk — meaning a chunk of the field's headline numbers may be detecting platform provenance, not deception.

**Key quotes:**

> "The adoption of embeddings and large language models has not translated into improved predictive performance."

The paper's cleanest falsification of the "newer model = better detector" assumption, and a useful counterweight to the default instinct to reach for the largest available model on narrow classification tasks.

> "Only 23.96% of models were evaluated on an independent sample… testing a model on new data reduces the predicted accuracy by up to 4.7 percentage points."

Three-quarters of the field's models have never been stress-tested on data they weren't trained on. In-sample and within-dataset evaluation is the deception-detection equivalent of grading your own exam.

> "Only models based on linguistic and statistical features could be directly connected to theories."

An argument for interpretability as a first-class requirement, not a nicety: in high-stakes deployment (courts, security), a 74%-accurate model whose features map to psychological theory may beat an 80%-accurate opaque one.

> "Human involvement tends to reduce the performance of an otherwise accurate deception-detection model."

The human-in-the-loop reflex, inverted: here the loop is the failure mode.

**Themes:** #concept (accuracy ceilings, ground-truth validity) #pattern (methodological confounds masquerading as progress) #tool (deception classifiers as a case study in evaluation discipline)

**Opinionated take:** This paper is a mirror for anyone building LLM systems, not just deception researchers. Its central claim — that methodological rigor dominates model choice — is exactly the pattern this wiki keeps encountering elsewhere: benchmarks that reward artifact detection ([[Goodhart's Law and AI Benchmarks]]), LLM classification that underperforms humble statistical baselines ([[Why LLMs Fail at Tabular Prediction]], [[LLM Classification Is Feature Extraction]]), and the general finding that smaller, purpose-fit models beat frontier generalists ([[When Smaller Models Win]]). The −1.2-point LLM penalty is particularly pointed: prompted frontier models are *worse* than logistic regression at a task they are popularly assumed to crush. The lesson generalizes — before crediting the model, audit the labels and the evaluation split. The paper's own prescriptions (shared corpora, pre-registration, model cards, pooled shared tasks) are the boring infrastructure that actually moves ceilings, and they apply mutatis mutandis to agent evaluation too.

**Related pages:**

- Strengthens [[When Smaller Models Win]] with the most systematic evidence yet that architectural sophistication buys little on narrow classification tasks — LLMs here lose to logistic regression outright.
- Complicates [[Goodhart's Law and AI Benchmarks]] by showing the corruption starts upstream of the benchmark: the labels themselves (non-verifiable, human-inferred, dataset-confounded) encode the reward before evaluation even runs.
- Nuances [[LLM Classification Is Feature Extraction]] from the opposite direction: that essay finds LLMs salvageable as feature extractors, while this meta-analysis finds raw prompted LLMs underperforming even simpler baselines — both agree the "LLM as classifier" framing is the broken part.
- Echoes [[Why LLMs Fail at Tabular Prediction]]: simple feature-based models holding their own against deep architectures on structured prediction, again.

---
*Sources: [[raw/2610-12118]], [[summary/2610-12118]]*
*Last updated: 2026-10-11*
