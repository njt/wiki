# LLM Classification Is Feature Extraction

A methodological essay arguing that using an LLM as a classifier — prompt in, hard label out — fails the basic desiderata of classical ML (calibration, covariate incorporation, interpretability), and that the fix is to stop treating the LLM verdict as a prediction and start treating it as a *feature*. Wrapping the verdict in a logistic regression recovers all three properties for free, and a SemEval-2018 irony-detection case study shows the approach reaching post-competition state of the art with nothing but a regression on LLM-extracted features.

---

## Key Quotes

> "LLMs-as-classifiers, prompts applied to a context and returning a label, suck to work with. This is especially painful because they often perform pretty decently."

The opening is the whole trap in two sentences. If LLM classifiers were *bad*, nobody would use them; the problem is that they're good enough to ship and bad enough to be undebuggable. The hard label hides everything you need to operate a classifier in production — no threshold, no confidence, no covariates.

> "Note that in the special case of β→∞ this basically recovers our LLM classifier!! But that's a dumb parameter selection policy."

The mathematical joke that carries the thesis: the LLM-as-classifier paradigm is logistic regression with the one coefficient set to infinity, chosen by vibes instead of data. Once you see the LLM verdict as *one input* to a fitted model rather than the answer itself, calibration, threshold control, and covariate blending all collapse into standard practice.

> "This is an arcane undertaking about which advice abounds on the internet but wisdom is scarce. Best of luck to you."

On prompt-tweaking as the only improvement lever in the LLM-as-classifier paradigm. The most quotable dismissal in the piece: under the feature-engineering frame, "make the prompt better" becomes the narrow, low-feedback activity it always was, while the real levers (more data, more features, better architecture) are the ones ML already knows how to pull.

> "Like we're going to need a test set in order to test model performance anyways (you were going to quantify your performance right?) so what's a little more for training?"

The honest acknowledgment of the paradigm's real cost. The entire allure of LLMs-as-classifiers is training-free deployment; this approach gives that up. The author's rebuttal is fair but breezy — if you weren't going to build a test set, this post has already failed to convince you.

> "We see that our initial LLM classifier beats the competition winner handily (0.747 vs 0.705)."

The empirical bait, and it's set carefully: the 2018 competition winner is beaten by a *hard label* from a cheap 2026 model, but the honest claim is narrower — the calibrated+featured pipeline (0.779) only *overlaps confidence intervals* with the post-competition SOTA (0.786). The author never claims to have beaten the LSTM-and-attention literature outright.

## Key Themes

#concept — LLM verdict as feature, not prediction; calibration as a property of the pipeline, not the model
#pattern — classical ML levers (data, features, architecture) replacing prompt-tinkering as the improvement loop
#tool — gemini-3.1-flash-lite batch classification with structured JSON output at temperature 0

## Analysis

The strongest move here is empirical, not rhetorical. The hard-label classifier posts a TPR of 0.965 — numbers like that get shipped — while its Brier score of 0.259 is barely distinguishable from a coin flip (0.25). That gap *is* the argument: the metrics people brag about (accuracy, recall, F1) don't expose miscalibration, and the metric that does is terrible. An LLM classifier looks great in a demo and is nearly unusable for anything that requires an operating threshold, because "Ironic" actually means "68.7% ironic" once fitted. The one-line logistic regression doesn't make the LLM smarter; it makes the LLM's existing signal *usable*, which is a different and more important thing.

Worth noticing what the results actually show, because the framing slightly outpaces them. Calibration changes no rankings — F1 stays 0.747 after fitting; every point of F1 gain (0.747 → 0.779) comes from the *features*, especially the 19 sub-questions. And those sub-questions were chosen by a human after eyeballing ten misclassified tweets. So the pipeline still runs on taste at the feature-ideation step; what the regression adds is discipline at the fitting step. The author's "screen our features" and "treat them as a secondary target" advice gestures at this loop but the post doesn't automate it. The closing idea of **agentic classifiers** — where the LLM investigates a context and its own investigative rigor becomes a feature — is the natural (if still speculative) attempt to push the LLM further down the feature-extraction path, into generating its own covariates rather than answering a fixed questionnaire.

There's also a quiet tension with the training-free promise. The framing gives up the one thing that made LLM classifiers attractive in the first place, and the "you needed a test set anyway" argument assumes a team that was already measuring. Teams that weren't — and the post's own examples (classifying ChatGPT conversations, slop investigations) suggest many aren't — now face label collection, split hygiene, and refitting on drift. The real thesis is narrower than the title: *if you are willing to do ML, you can have the LLM's world knowledge and your calibration too.* That's a genuinely useful result, and the convergence with the 2024 feature-generation literature suggests it's the direction the field is actually going.

## Related Pages

- [[Why LLMs Fail at Tabular Prediction]] — a complementary diagnosis pointing at the same conclusion from the other direction: Garnelo & Czarnecki isolate dimensionality as what breaks LLM prediction on structured data, while this post shows the escape route — don't ask the LLM to be the predictor, use it as a feature extractor feeding a classical model that handles the structure.
- [[LLM Evals]] — this post is a concrete worked example of that page's core discipline: the Brier-score revelation (0.259 ≈ random) is exactly the failure mode you get when you evaluate a classifier only on the metrics that flatter hard labels, and calibration is the property evals should have been measuring all along.
- [[Fine-Tuning a Local LLM to Categorize Questions]] — two field reports on LLM classification that take opposite roads to fixability: Helgevold retrains the model (and finds even the *label encoding* matters), while this post keeps the model frozen and moves all the learning into a wrapper regression; the shared finding is that uncalibrated raw LLM verdicts are not the end of the pipeline.
- [[LLM-as-a-Verifier]] — both reject the hard discrete verdict in favor of richer signal from the same forward pass: that paper builds continuous scores from scoring-token logit expectations, this post harvests logprobs, subverdicts, and multi-run features; both treat the model's first-token answer as a wasted opportunity.

---
*Sources: [[raw/llm-classification-is-feature-extraction]], [[summary/llm-classification-is-feature-extraction]]*
*Last updated: 2026-09-19*
