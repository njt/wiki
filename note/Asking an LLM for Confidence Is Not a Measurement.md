# Asking an LLM for Confidence Is Not a Measurement

rumple's essay dismantles the common agent-design habit of asking a model "how confident are you?" and using the number as a routing or escalation signal. A trustworthy confidence score needs two things — it must reflect the evidence behind a prediction, and it must have been calibrated against recorded outcomes on the same kind of question. An LLM's self-reported number has neither, because it is a token-by-token continuation of text, not a read-out of internal certainty. The constructive alternative: compute confidence from observable signals and a record of graded outcomes, outside the model.

---

## The argument

The essay first pins down what confidence *means* using a weather forecaster: confidence is derived from information (evidence about tomorrow plus base rates from the past), and two forecasters with the same information applying the same rule must reach the same number. Accuracy is then only checkable over many days — a 70% forecast is "right" if it rains on about 70% of the days the forecaster said 70%. That is *calibration*. But calibration alone is insufficient: a forecaster who always says the seasonal base rate of 30% is calibrated yet useless, because the number carries no information. And underneath both sits an unglamorous requirement: someone must write down every forecast and what happened.

Against that standard, the LLM's number fails twice:

- It does not reflect the information, because it is "a plausible continuation of the conversation", not a summary of the evidence.
- It was never calibrated on confidence. Pretraining scored the model only on "what text comes next" — the model "learned what confidence scores look like in text. It did not learn to make its scores match its accuracy."

The falsifiable symptoms follow: the same question yields 0.85 one time and 0.7 the next; "Are you sure?" moves the score despite adding no information; and text already written ("This is clearly the right answer") decides the number that follows. The number always appears and always looks precise — and, as with summaries, "an incorrect number looks the same as a correct one."

## The constructive recipe

For a customer-support agent where uncertain replies get routed to a human, the essay proposes building the confidence outside the model in five steps: (1) list observable signals — did search hit an article, is this a question type seen before, do two sampled replies agree; (2) start from the historical correct-rate (75% if 150 of 200 graded replies were correct); (3) record every reply, its signals, its score and its outcome; (4) update the rule by grouping past replies by signals and using each group's empirical correct-rate; (5) re-check the calibration table periodically and fix overconfidence. The resulting rule is deterministic given the same information, depends on evidence not wording, and is calibrated on *your* kind of question — but only if someone actually grades the replies.

## Key quotes

> "The model writes it, token by token, in the same way that it writes any other text. '0.85' is a plausible continuation of the conversation. It is not a measurement of anything."

The mechanistic core, and the cleanest one-sentence rebuttal to confidence-prompting. It also explains why the failure is structural rather than a prompting problem: no prompt turns an autoregressive sampler into a probability gauge.

> "The model learned what confidence scores look like in text. It did not learn to make its scores match its accuracy."

This separates *format learning* from *calibration*, a distinction most agent designers blur. The model is genuinely calibrated — on next-token prediction, which is the wrong outcome variable entirely.

> "You can find out whether the numbers are trustworthy only by grading many of them against an answer key. Most people who ask an LLM for a confidence score do not do that."

The epistemics land on the record, not the model. This is the same insight the essay applies to its own recipe: step 5's calibration table is worthless without step 3's grading discipline.

## Themes

#concept #pattern

## Analysis

This is a rare essay that gets the epistemology of confidence *exactly* right while staying short and constructive. The three-part standard — evidence-responsiveness, calibration on outcomes, and the existence of a record — is a genuinely useful rubric you can apply to any "the agent said it was 90% sure" claim. Its dismissal of verbalised confidence is correct but worth sharpening: it does not mean models have no usable internal uncertainty signal (logprobs, sampling consistency, and verbalised-confidence research all exist); it means *the number the model types* is not that signal. The essay's recipe is essentially building a poor-man's calibrated classifier over agent-behavioural features, which is the right move — though it quietly inherits the hard part, since "was the reply correct?" needs its own grading pipeline, and graded records are exactly the thing most teams do not keep.

The deepest implication is for agent *design*, not just agent *evaluation*: any agent architecture that uses self-reported confidence to gate escalation (human handoff, tool selection, spend) has installed a random number generator at a decision point and named it a gauge.

## Related pages

This strengthens [[Jev Can't Be Calibrated]] from the other direction: Molas argued a model cannot be calibrated for every customer distribution at once and needs local recalibration; this essay argues an LLM's verbalised score was never calibrated anywhere at all, since the training outcome was next-token text — together they bracket calibration as a property of model-plus-record, never of the model's prose. It complements [[The Lifecycle of LLM-as-a-Judge]], which shows what a properly maintained judge looks like — a training signal keyed to outcomes and standing drift monitoring — precisely the properties the self-reported confidence score lacks. It nuances [[Agentic Verification]]'s "build loops the agent can run itself" thesis by pointing out that one popular loop input, the agent's own confidence, is fake; verification signals must come from observable behaviour and outcome records instead. And it gives a concrete mechanism to [[Decision Models for Evals (Hamel)]], whose decision-model framing for evals starts from exactly the base-rate-plus-record logic this essay derives from weather forecasting.

---
*Sources: [[raw/asking-llm-for-confidence-html]], [[summary/asking-llm-for-confidence-html]]*
*Last updated: 2026-10-24*
