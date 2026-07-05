# Fixing LLM Writing with Distribution Fine Tuning

Ben Rosmine demonstrates that standard SFT systematically fails to match the distribution of training data, and proposes Distribution Fine Tuning (DFT) — a proprietary post-training algorithm that optimizes output *distributions* rather than individual samples. A 14B DFT model achieves JMQ 0.80 (80% of the way to indistinguishable-from-human), +164% creativity, and eliminates characteristic LLM "slop signs" like emdash overuse. The work is technically impressive but structurally unverifiable: proprietary algorithm, no paper, no open weights, a demo with deliberately injected errors to prevent evaluation.

---

## Key Quotes

> "Slop. It's not just annoying — it's exhausting."

The opening line that names the visceral reaction driving this work. Rosmine gets that slop isn't a technical problem for most readers — it's a felt experience of being fed content nobody cared enough to write.

> "SFT focuses on training individual samples and in doing so misses out on distribution level information."

The core insight. SFT optimizes `P(token | context)` per example but never sees the aggregate shape of its output distribution. This is a genuinely important observation that connects to known problems (exposure bias, miscalibration, local typicality) but names the unifying failure mode: the *distribution* is the thing you're trying to match, and per-sample loss doesn't get you there.

> "LLMs are not the cause of slop. Lack of effort/care is."

Controversial and worth wrestling with. The claim is that a well-researched, detailed outline + LLM prose = good writing, even with stylistic tics. This is half right — effort matters enormously — but it understates how much LLM defaults shape *what kind* of effort people put in. The emdash habit isn't laziness; it's what the model does when you *don't* fight it.

> "Right now, LLMs for writing are like GPT4 for coding. People think that LLMs help them write, but it's actually just adding bugs faster."

The sharpest line in the piece. An honest admission from someone building LLM writing tools that the current state is net-negative for quality. The "IDE for writing" vision — unit tests for prose, fine-grained control, automated quality checks — is the right ambition.

> "The demo was trained on a subset of fineweb, a collection of web documents. This should make the demo models good at blogs/news articles, but it is unlikely to do as well at creative writing."

Refreshing honesty about scope. Most ML papers would gloss over this. Rosmine explicitly bounds what the demo can do.

## Key Themes

- **#concept Distribution-level training**: The core technical contribution — optimize the shape of the output distribution, not per-token prediction. This is a genuinely underexplored direction in LLM post-training. [[Where the Goblins Came From]] shows what happens when you *don't* do this (reward models create paperclip maximizers for playful metaphors).

- **#tool Metrics for writing quality**: MMD, JMQ, and L2 Token Distance form a three-axis evaluation framework. Not perfect — JMQ is judge-model-dependent and judge models prefer LLM outputs (Laurito, Panickssery) — but more systematic than "vibes." [[StoryScope]] offers a complementary approach: 304 narrative features that distinguish human from AI fiction at 93.2% F1.

- **#pattern Super baseline methodology**: Max-over-all-hyperparameters for the baseline, single fixed config for DFT. This is clever experimental design that makes the baseline artificially strong — but it's also a concession that the DFT advantage might be narrower than the headline numbers suggest. If you need to construct a super-baseline to show your method wins, the real-world gap might be smaller.

- **#pattern Proprietary algorithm, public claims**: DFT is closed — no code, no paper, no weights. The demo injects random words to prevent copy-paste evaluation. This creates an uncomfortable dynamic: the claims are specific enough to be falsifiable but structured to prevent independent falsification. Not necessarily bad faith (building a business), but it means everything here should be read as marketing-adjacent.

- **#concept Slop as distributional failure**: Reframes "LLM writing is bad" from a vibes complaint to a measurable distributional mismatch. This is the right level of abstraction — it turns an aesthetic argument into an engineering one. [[Why Does AI Write Like That]] covers the aesthetic side; this covers the measurement side. [[Various LLM Smells]] covers what the user experiences.

## Critical Analysis

**The metrics are clever but the evaluation has a circularity risk.** MMD and L2 Token Distance measure similarity to the training data distribution. If you optimize directly for these metrics, you're doing something closer to domain adaptation than quality improvement. JMQ is the only independent quality signal, and it's judge-model-dependent. The fact that GPT-5.4 scores DFT higher doesn't tell us whether humans would agree. The Pangram "100% human written" claim is eye-catching but Pangram is a commercial detector, not a peer-reviewed benchmark.

**The super-baseline is a double-edged sword.** It makes the DFT win more impressive on paper, but it also means we're comparing a single DFT model against a theoretical ceiling for SFT that no actual SFT model achieves. If I deploy SFT at T=0.7 and DFT at its fixed config, the real gap depends on which SFT config was actually best for that metric — not the max over all configs. The graphs show DFT beating SFT at every temperature, which is the stronger claim.

**The BLEU decline deserves more scrutiny than it got.** DFT's BLEU score is *worse* than SFT (0.051 vs 0.062 for 14B). Rosmine argues this is because SFT overuses common grammatical patterns that inflate BLEU. That's plausible — BLEU is a terrible metric for open-ended generation — but it's also the kind of result you'd see if DFT were producing more *varied* but not necessarily *better* text. The self-BLEU diversity numbers partially address this.

**185K samples is tiny, and the SFT plateau at this size is suspicious.** Rosmine shows SFT metrics plateauing (or regressing) from half to full data. If true, this means either the Fineweb subset is saturated at this size, or something about the training setup (LoRA rank? learning rate schedule?) is limiting SFT. Either way, it raises the question: would DFT's advantage shrink if you trained on 10× the data with full fine-tuning? The claim that "DFT can get us improvement much more cheaply" is probably true, but cheap improvement on a 185K-sample budget might not translate to frontier-scale training.

**The anti-slop mitigations are genuinely thoughtful.** Requiring outlines, injecting copy-paste detection, no public API — these aren't just responsible-AI checkbox items. They're designed by someone who understands both the abuse vector and the user psychology. The "unit tests for writing" vision is the most interesting product idea here, and it's frustrating that the post gestures at it without details.

**Bottom line**: If DFT works as described, it's a significant advance in post-training methodology. The core insight — optimize distributions, not samples — is important regardless of whether DFT specifically is the right algorithm. But this is a proprietary claim from a solo researcher with a GPU server and a business to build. The right posture is "fascinating, hope it's real, need independent replication." The 14B JMQ of 0.80 is the kind of number that, if it holds up, would make this the most important writing-model paper of 2026. But we've been here before with unverifiable ML claims.

---
*Sources: [[raw/fixing-llm-writing-distribution-fine-tuning]]*
*Last updated: 2026-07-05*
