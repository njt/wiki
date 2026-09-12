# Stop Anthropomorphizing Intermediate Tokens

Kambhampati et al. argue that the field's habit of calling intermediate tokens "reasoning traces" or "thinking traces" is a dangerous anthropomorphization, not a harmless metaphor: trace semantics are largely decoupled from answer correctness, trace length is not "thinking effort," and showing traces to users *increases* false trust rather than transparency. Their remedy is to treat traces as learned prompt augmentations — scaffolding for the model, not windows into the model — and to verify solutions directly.

---

## Key Quotes

> "While a human may say 'aha' to indicate exactly a sudden internal state change, this interpretation is unwarranted for models which do not have any such internal state."

The "aha moment" dissection is the paper's sharpest single jab. DeepSeek R1's paper celebrated the model emitting "aha" as evidence of a spontaneous realization — but the model has no internal state that the token could report on. On the next forward pass, the only difference from the pre-"aha" pass is that one extra token sitting in the context. The word is doing work *as a token*, not *as a report*.

> "What matters for a model to improve accuracy with intermediate tokens is not their semantic import, but perhaps a consistent pattern in the training data for the model to fit itself."

This is the empirical heart. Their maze experiments show that models trained on A* search traces keep improving even when trained on *swapped* (nonsense) traces — accuracy dips only when correct and swapped traces are *mixed*. What the model needs is a stable, consistent scaffold to fit against, not a semantically valid derivation.

> "Reasoning traces and their summaries increase user trust in the model's predictions regardless of their correctness, thus substantially increasing false trust."

The false-trust finding (Palod et al. 2026) is the most consequential for product design. Showing the "thinking" makes users more likely to accept *wrong* answers — the exact opposite of the transparency rationale vendors give for exposing traces. In settings where users can't independently verify the answer, the trace is a confidence exploit, not a safety feature.

> "Any human interpretable meaning in the intermediate tokens might be a fortuitous coincidence of such rationales being present in the training data, rather than the model actually doing internal computations corresponding to those statements."

The clearest statement of the paper's central distinction: correlation is not causation, and plausibility is not mechanism. Models trained on human text will emit human-sounding "reasoning" because human reasoning is in the training data — not because the model is reasoning.

## Key Themes

- **#concept intermediate tokens vs. derivational traces** — The authors coin "derivational trace" as a neutral stand-in for the unfiltered pre-answer tokens, to escape the loaded "chain of thought" terminology. Crucially they distinguish these from *post-facto rationalizations* (OpenAI's sanitized o1 summaries) and from *agentic tool calls*, which must carry external semantics.
- **#concept trace–answer decoupling** — The empirical claim that intermediate-token validity and final-answer correctness are only loosely coupled. Wrong, swapped, or truncated traces still improve accuracy; correct traces don't guarantee correct answers.
- **#pattern length ≠ effort** — Trace length is not a measure of problem difficulty or thinking effort. RL with uniform reward credit structurally incentivizes longer traces (Samineni et al. 2025), so "learning to reason = longer traces" was a credit-assignment artifact.
- **#pattern prompt augmentation** — The proposed alternative framing: intermediate tokens are a learned Skolem function that maps each task to a context augmentation that pulls the instance toward the model's training distribution. No interpretability required — and the adversarial-jailbreak literature (gibberish suffixes, latent-space tokens) already shows augmentations need not be human-readable.
- **#person Subbarao Kambhampati** — The ASU professor whose group (Valmeekam, Palod, Bhambri, Samineni, Stechly, Kalwar) has spent years arguing LLMs "can't plan, but can help planning" via LLM-Modulo, and now extends that skepticism to reasoning models' traces.

## Critical Analysis

**The strongest part is the causal intervention, not the polemic.** Position papers are cheap; swapping A* traces between maze instances and watching accuracy hold is a real result. If models trained on *semantically wrong* traces still improve, then the entire "reasoning trace" research program — process reward models, faithfulness benchmarks, "monitorability" appeals — is building on a correlation the authors have shown to be optional. That's the argument no "but users like seeing the thinking" reply can touch.

**The prompt-augmentation view is elegant but still speculative.** The authors are honest that it's a "speculative future direction." What they don't fully confront is that a scaffold-that-happens-to-look-like-reasoning may be doing *more* than fitting to the training distribution — the latent-programming and global-workspace work suggests intermediate activations do carry structured computation even when the *surface* tokens are decoupled from it. The correct conclusion may be "the tokens aren't the reasoning," not "there is no reasoning." [[Fractal Basins Trap Latent Reasoning]] supplies mechanistic evidence for the "there is reasoning" side: decoding the *latent* trajectories of reasoning models shows they stall near nearly-correct solutions (Sudoku grids with repeated digits, maze dead ends) before backtracking — real problem structure in the hidden state, even where the surface tokens are decoupled from it.

**The false-trust result is the paper's real product warning.** It reframes trace display from an ethics nicety into a liability: if showing traces increases acceptance of wrong answers, the vendors that *hide* them (OpenAI, Google, Anthropic — ostensibly for proprietary reasons) are accidentally doing the safety-correct thing, while DeepSeek R1's full-trace transparency is the one actively misleading users. That inverts the usual open-vs-closed moral valence in a genuinely interesting way.

**Where it's vulnerable:** the authors' own evidence mostly comes from *their group's* studies — a coherent, mutually-citing cluster. That's fine for a position paper but means the "collating significant body of emerging work" is partly self-referential. The independent confirmations they cite (Li et al. 2025a, Su et al. 2024's Dualformer, Chen et al. 2025) are doing real work, and the argument would stand or fall on those, not on the maze experiments.

**How it sits in the wiki:** This is the [[A Non-Anthropomorphized View of LLMs|Flake position]] applied specifically to reasoning traces — Flake says don't treat the *model* as a mind; Kambhampati says don't treat the *trace* as a mind's work. It complicates [[Controlling Reasoning Effort in LLMs|Raschka's effort-control survey]], which takes "trace length = effort" as a usable knob — the authors argue the knob is measuring a credit-assignment artifact. And it pairs with [[The Reasoning Trap]], which found reasoning RL amplifies tool hallucination: if traces are semantically meaningless scaffolding, it's exactly what you'd expect a model to do — generate plausible filler — when asked to "think" harder.

## See Also

- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake's anti-anthropomorphism applied to the model itself; this paper is the trace-level complement
- [[Controlling Reasoning Effort in LLMs]] — Raschka's survey treats reasoning-effort/length as a controllable knob, which this paper argues is measuring a credit-assignment artifact
- [[The Reasoning Trap]] — reasoning RL amplifies tool hallucination; the trace-as-scaffolding view explains *why*
- [[Anthropomorphism in Children's Interactions with LLM Chatbots]] — anthropomorphism operates at the interface regardless of substrate; the false-trust result is the adult analog
- [[LLM-as-a-Verifier]] — the paper's recommended remedy: trust should come from verification of the answer, not the trace

---
*Sources: [[raw/2504-09762v4]], [[summary/2504-09762v4]]*
*Last updated: 2026-09-04*
