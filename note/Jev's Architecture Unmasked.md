# Jev's Architecture Unmasked

Archer Hume's black-box forensic reconstruction of Jev, TypeSafe's decision model: ~10,000 API calls, 1,029 instrumented probes and a published evidence bundle yield the most credible public account of how the "System One" model works — a causal (likely sparse-MoE) transformer that stops after prefill and reads decision probabilities straight off its hidden states, serves many isolated question branches from one shared state KV prefix, processes options listwise rather than independently, and was trained (RLCD) so its distributions are calibrated against outcomes. Where the launch announcement made claims and the teardown made arguments, this source generates evidence — and labels every conclusion published, observed, or inferred.

---

## Key Quotes

> "Black box APIs make it shockingly easy to throw a blanket over the ghost and get a rough shape of what the architecture looks like."

The methodology in one line, against the reflex that closed APIs are opaque. Latency scaling, token accounting, and behavioural interventions across API boundaries constrain the design space hard enough to sketch it — with the honesty that what you get is a shape, not a schematic.

> "A generated '91%' is a token sequence. A classifier's 0.91 is an entry in its predictive distribution. Either can be miscalibrated. Neither becomes trustworthy solely because of its format."

The essay's epistemic core, and a quiet corrective to both Jev's marketing and its detractors. "Can't hallucinate" guarantees a format; calibration is the property that makes a number usable, and no format buys it.

> "The output no longer needs a sequence of spelling decisions for 'payments': 0.91. JSON formatting happens in ordinary application code. The neural network supplies the probabilities."

The computational-shape argument for why the decode loop is pure overhead on decision tasks: spelling out a decimal is work the application never needed done. Causal attention describes which positions can see which information, not the order input must execute in — prefill was always parallel.

> "It would be a mistake to divide that number by request duration and call the result the model's decoding speed."

Measurement discipline on display. The API's `output_tokens` field is a post-hoc billing figure — 4 shared tokens plus 15 per answer plus the identifier's length, matching none of 192 public tokenizers, uncorrelated with latency — and reading it as evidence of text generation is exactly the inference its existence invites and the data forbids.

> "Inject[ed] fake options never displaced the real ones, so option boundaries are marked in a way text cannot forge."

A security-flavoured finding from the option experiments: whatever separates options inside the model is not a plain-text marker, so prompt-level injection cannot create, merge, or displace options.

> "A threshold near 0.9 could change the action even though the labels and evidence are identical. Permutation tests belong in the evaluation of any implementation of this design."

The most operationally consequential finding: reversing option order moved a technical-support classification probability from ~0.84–0.89 to ~0.93–0.96. For any deployed policy branching near a threshold, option order is a hidden decision variable that calibration training does not obviously remove.

## Key Themes

- #concept — **Calibrated probability as the product.** RLCD targets proper scoring rules (log loss, Brier); bin-level ECE was 0.0313 on 1,200 MMLU items; the API's `confidence` field is arithmetic — c = (p_max − 1/K)/(1 − 1/K) — not a second learned estimate. Keeping the learned distribution and the derived summary separate prevents the classic error: a concentrated distribution can still be confidently wrong.
- #pattern — **Prefill-only decision serving.** Shared state prefix + isolated question suffixes packed into one sequence (≤2¹⁵ tokens per branch, ≤2¹⁶ per request, state counted once); prior art in Hydragen and DeFT. "Parallel" means scheduling with no answer dependencies — if a later question truly needs an earlier answer, the application must add a decision stage.
- #pattern — **Listwise option processing with anti-forgery boundaries.** Options influence each other (replicated fifth-option log-odds shift, +0.38 → +0.11, every block decreased), a reference card placed after the options is used (16/16), fake options are ignored. Slot-head vs pointer readout left deliberately open.
- #tool — Jev (jev-1.13.0) and the TypeSafe API, probed via latency headers, additive token accounting, and tokenizer fingerprinting against 192 public tokenizers (closest match Qwen at 348/415; o200k-adjacent but ruled out — a replaced vocabulary, continued pretraining, or distillation could each explain it).
- #person — Archer Hume, and a meta-method worth copying: every claim labelled published / observed / inferred, falsifiable predictions (the log-odds invariance test) stated before the experiments that test them, and limits disclosed (upstream latency headers, two-decimal precision, correlated duplicate questions, one account in one region).

## Opinionated Take

This is the best of the wiki's three Jev sources because it is the only one that generates evidence rather than argument. The launch note told us what TypeSafe claims; the teardown told us what a skeptic could build in twenty lines; this tells us, with controls, blocks, and repetitions, what the black box actually does. The secret-code intervention (move one sentence across an API boundary and watch p move from 0.00 to 0.92) is a model of what a well-designed probe looks like: an intervention, a control, and a falsifiable prediction stated in advance.

The method is arguably the real deliverable. Closed AI products invite two failure modes — believing the marketing, or dismissing it — and both are lazy. Hume's template (token-accounting forensics, behavioural interventions, ranked certainty over conclusions) is a reusable discipline for auditing any closed system, and the essay's repeated self-limitation ("a separate model call... could produce the same result", "this remains an inference, not a measurement") is precisely what makes it trustworthy rather than merely confident.

Where it stays genuinely unsettled: sparse MoE is a plausibility argument, not a measurement; the slot-vs-pointer readout is explicitly open; and the option-interaction mechanism could be shared temperature or content-dependent mixing. The calibration numbers are bin-level but from one 1,200-item sample with most predictions piled near certainty — encouraging, not conclusive. And the fresh-math gap (86.7% on multiplication, 32% on two-step word problems) is a useful reminder that a public benchmark score measures a sampling of what a model knows, not the knowledge itself.

The finding that should change practice is the least glamorous one: option order moved probabilities across a plausible 0.9 threshold. Every "calibrated decision" API inherits format- and ordering-level sensitivities; if you consume such an API, permutation tests are not optional.

## Related Pages

- [[System One Models and Jev]] — Strengthens the launch note by giving its marketing claims an empirical substrate: "parallel probabilities" and RLCD stop being assertions and become measured behaviour (isolation probes, ECE 0.0313), and the earlier note's open question about what RLCD actually buys gets a partial answer — along with the deflating datum that the `confidence` field is arithmetic, not learned.
- [[You Could Have Built Jev]] — Complicates the teardown's demystification. "Twenty lines of pseudocode" holds for deleting the decode loop, but Hume's evidence — listwise option interaction that defeats independent-logit explanations, anti-forgery option boundaries, a tokenizer unlike any public one — locates whatever moat remains in exactly the places the clone experiments could not check.
- [[Holding the LLM Stack in Your Head]] — The forensic application of that stack literacy: prefill vs decode, KV cache economics, tokenizer fingerprints. Gustafson's series teaches the layers; this essay shows the same layers used as instruments against a closed product.
- [[LLM-as-a-Verifier]] — Sibling move, opposite direction: both reject sampled text as the interface to model belief and read the distribution instead. The verifier's logit-expectation scoring and Jev's readout head are technical cousins, and Hume's calibration data is the kind of evidence the verifier literature rarely reports about itself.

---
*Sources: [[raw/jevs-architecture-unmasked]], [[summary/jevs-architecture-unmasked]]*
*Last updated: 2026-09-22*
