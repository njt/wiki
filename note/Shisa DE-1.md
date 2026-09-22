# Shisa DE-1

Shisa AI's open-weight Apache-2.0 "decision model": a Gemma 4 26B-A4B fine-tune whose training loss is identical to its deployment readout — restrict the next-token logits to the option-letter rows at the answer boundary and target the correct letter — so every answer is a typed choice or a Yes/No probability read straight from the distribution, never generated text. The card is also a small manifesto for a model class: bigger LLMs generalize better than small specialist classifiers, still run in real time (20 ms p50 on 2×H20-3e), and can honestly publish what they cannot do.

---

## Key Quotes

> "Instead of an answer text, each answer is read from the next-token distribution at the answer boundary, and restricted to the option letters."

The one-sentence definition of the model class, stated as an architectural fact rather than a serving trick. DE-1 never emits prose; the interface is the distribution itself, which is why the card can promise typed answers without a parser and without the "models like to chat" failure the Jev clones had to engineer around.

> "The loss is the deployment readout: the next-token logits are restricted to the K option letter rows and the cross-entropy targets the correct letter."

The card's real contribution, in one clause. Nearly every LLM-classifier story in this wiki trains on generated text and serves something else; here train objective and serving contract are the same tensor operation, so the 0.888 → 0.936 → 0.968 training curve is measured in the exact units production consumes.

> "Tokens outside the option list are discarded even when they outrank a valid option, as `D` does here. `choices[0].text` is the sampled token, shown for debugging only; the contract reads the distribution, not the sample."

The readout contract made explicit in the worked example (a phishing email asking for card details → "payment card details" at 0.9973). It is also a quiet rebuttal of sampling: the API's returned text is demoted to a debugging aid, and the seventh-ranked invalid token `D` is discarded by policy, not by luck.

> "A dot-product head over frozen hidden states was also trained and scored lower on the sealed compaction test: 0.837 AUROC at its best epoch against 0.8902 for this model."

The ablation that justifies the whole design. "No task head is added to the graph" is not an aesthetic choice about causal LMs — training the LM's own readout beats training a conventional classifier head over frozen features by five AUROC points. The distribution *is* the head.

> "While even much smaller classifier models can perform admirably at many decision-making tasks, our testing showed that larger LLMs provide better generalization and can still run within real-time performance windows."

The comparative thesis, backed by a table where 150M–576M specialists (ModernCE, GLiNER, BGE-M3, Laya) collapse to 0.3–0.8 on the decision suites while DE-1 holds 0.9+ at 20 ms p50. The gap between "admirable" and "deployable" is generalization, not peak accuracy.

> "The readout is not scoring the option text alone."

The card's most uncomfortable sentence, delivered after the option-order-reversal experiment: on identical cases and gold, the model picks count 3 on 22 ascending presentations and 40 descending ones. Position bias survives readout-restricted training — the typed answer is partly an artifact of presentation order, not just of the state.

> "No generation quality is claimed. Only the restricted-letter readout was trained and measured. Open-ended generation, chat, and tool use are unsupported by the evidence here, and the vision and audio paths were not evaluated at all."

A model card that declines to claim what it did not measure — the vision encoder ships in the weights, untouched, and the card says so. This is the honesty standard most release posts fail, and it makes the reported numbers more credible, not less.

## Key Themes

- **#concept — Readout-as-loss decision models.** A model class where the answer interface (restricted option-letter distribution at the answer boundary) is also the training objective, eliminating the train/serve gap. The name "Decision Engine" claims System One territory.
- **#pattern — Contract-first serving.** One request per question, `max_tokens=1`, single-token letter requirement (capping K at 26), a documented fallback (`prompt_logprobs: 0`) when a letter misses `top_logprobs`, and the sampled token demoted to debug output. The contract even specifies what is *not* part of it.
- **#tool — Open-weight serving economics.** Mainline vLLM, 2×H20-3e TP2, a 596–684 req/s plateau across concurrency 64–512, and 129,429 MiB per GPU — the throughput/latency tables are the self-hosting case against a hosted baseline.
- **#concept — Calibrated-by-refit confidence.** Post-hoc temperatures fitted per answer kind (T_noul 1.69, T_choice 1.90), choice ECE 0.138 → 0.046 on n=7,446, with leave-one-suite-out fits spanning 1.76–1.89 — one temperature transfers, but the card insists you refit on your own rows before trusting the probabilities.

## Analysis

The card is best read as the trained answer to the question [[You Could Have Built Jev]] left open. That teardown showed the single-token readout trick is twenty lines of pseudocode and asked whether TypeSafe's moat was secret architecture, RLCD training, or parallel sampling. Shisa DE-1 is a fully documented implementation of the same class — its hosted baseline is literally Jev ("baseline this model replaces"), its eval suites include the openjev clones' naming lineage — and its answer is: no hidden architecture, just train the readout itself. The interesting finding is that training the readout is what buys the wins over the base model (guardrails 0.917 vs 0.833, fraud 0.999 vs 0.985) and over every small specialist, while the technique alone was already free.

The fine-tuning gains are narrower than the headline suggests, and the card mostly admits it. On the base model the same readout already scores 0.976 Japanese routing and 0.895 ag_news — training moved neither. It *lost* ground on ja-indirect (0.687 vs base 0.896, and below Jev's 0.844) and on compaction it loses to Jev outright (0.859 vs 0.923). Held-out decision families are unsolved (0.491 vs 0.810), and the prompt-scaffold A/B that moved nothing rules out a rendering fix — this is a readout limit. So the honest summary of the table is: readout-restricted training sharpens the families you train and can actively blur ones you didn't, which is the standard fine-tune bargain priced in public for the first time in this model class.

Two results here travel furthest beyond this one model. First, the dot-product-head ablation (0.837 vs 0.8902 AUROC): the LM readout beat a conventional classifier head over frozen hidden states, which is evidence for a claim that cuts against the entire "LLM as feature extractor" school — the pretrained distribution over answer tokens is a better head than anything you bolt on afterward. Second, the option-order result: mean absolute shift 0.029 sounds small until you see the 11-option counting probe flip 22→40 on identical cases. Typed decisions are not automatically position-invariant decisions, which complicates the "can't hallucinate, just reads probabilities" marketing of the whole class ([[System One Models and Jev]]): the guarantee covers answer *shape*, and the distribution still reads the layout.

The structural ceiling is worth naming too. The single-token letter contract caps a question at 26 mutually exclusive options, so the Banking77 suite (77-way intent) is a partial run for DE-1 and the base model, while hosted Jev answered all 776. "General-purpose decision model" is really "general-purpose decision model up to 26 options per question" — a real limit on the claim, and one the card handles by reporting the partial run rather than hiding it.

## Related Pages

- [[You Could Have Built Jev]] — the direct ancestor and the card's named baseline: that teardown asked whether the single-token readout trick had a moat beyond the technique, and this card answers it with the trained version — the moat is the training objective matching the readout, not a secret architecture; it also supplies the Jev-vs-open comparison data the teardown said could not be run.
- [[System One Models and Jev]] — TypeSafe's launch of the decision-model class DE-1 joins: this open-weight entry strengthens that page's claims with published training and eval data, but nuances the calibration story (temperatures must be refit per deployment) and complicates the "no hallucinations" framing with the option-order finding.
- [[LLM-as-a-Verifier]] — the same "read the distribution, not the sample" move pushed one step further: the verifier paper reads logit expectations from a general model, while DE-1 trains the restricted readout itself, making the distribution-native interface the optimization target rather than a post-hoc scoring trick.
- [[LLM Classification Is Feature Extraction]] — the friendly opposition: that essay fixes uncalibrated LLM verdicts by demoting them to features in a fitted regression, while DE-1 calibrates the readout directly (post-hoc temperature, ECE 0.138 → 0.046) and its dot-product-head ablation suggests the pretrained distribution is already the better head — the two approaches disagree about where the classical ML belongs.

---
*Sources: [[raw/shisa-de-1]], [[summary/shisa-de-1]]*
*Last updated: 2026-09-22*
