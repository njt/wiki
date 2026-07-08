# Global Workspace in Language Models

Anthropic's interpretability team identifies a "J-space" — a sparse, privileged subspace of a language model's activations that acts as a functional global workspace: concepts posted here are available for verbal report, flexible reasoning, and directed modulation, while the vast majority of processing (parsing, fluency, continuation) routes around it. The technique, called the Jacobian Lens (J-lens), is the first interpretability tool that surfaces *what the model is poised to say* rather than what it happens to be saying. The paper doesn't claim consciousness — but it demonstrates that models have an internal "center stage" where abstractions compete for access to output, and ablating it produces a recognizable deficit: the model can still parse and classify, but loses the ability to hold intermediate concepts in mind, plan ahead, or describe experience in anything but mechanical terms.

---

## Key Quotes

> "The J-space component of a concept's representation, despite accounting for a small fraction of its variance, is responsible for that concept's availability for verbal report."

This is the paper's central empirical claim, and it's a striking number: the workspace component accounts for median 6–7% of activation variance, but swapping it changes the model's answer on 59% of trials. Swapping the *other* 93% changes the answer on only 5% of trials. The model's representational budget is wildly disproportionate — a tiny fraction of compute determines what it can talk about.

> "Many computations, which we might call 'automatic,' do not causally route through the J-space."

The selectivity result is what separates this from "the model has concepts." The same Spanish-language information that lets the model write fluent Spanish continuation and detect French intrusions (automatic processing) also appears in J-lens readouts — but swapping the J-space representation only changes the answer when the task *asks about* the language. The workspace isn't "everything the model knows." It's "what the model is prepared to deploy flexibly." This is global workspace theory's central prediction, confirmed in a transformer.

> "The model can parse text, classify it, and extract spans from it with the J-space suppressed. However, it loses its ability to assemble abstract characterizations of context and flexibly generate content that depends on them."

The ablation battery is the paper's most actionable finding. MMLU multiple choice, sentiment classification, CoLA acceptability, SQuAD extractive QA — all survive J-space ablation. But Caesar-cipher decoding, analogy completion, summarization, translation, sonnet writing — all collapse. The workspace isn't needed for routine linguistic processing. It's needed for the kind of reasoning that separates Claude from a very good autocomplete.

> "Under J-space ablation, the model writes fluently about its own processing, but the language of its reports changes to become more detached and mechanical."

The experiential-language experiment is the most provocative and the most restrained section of the paper. Ablating just the top-10 J-space directions in the workspace layer range causes a "dramatic" drop in experiential language score — the model stops describing things *as though experienced* and shifts to event-log prose. This holds across three model sizes, survives matched-norm controls, and extends to descriptions of *other people's* experiences, not just the model's own. The authors don't claim this is ablating consciousness. But they've found a manipulable neural correlate of experiential language, and it sits in the same subspace that handles intermediate reasoning.

> "The model in some sense 'thinks in English' in its intermediate layers."

The cross-lingual finding — Chinese prompt, English intermediate representations, Chinese output — is the kind of detail that makes interpretability papers worth reading. It's not the headline result, but it's the kind of empirical fact that constrains theory: whatever the workspace is doing, it's doing it in the model's dominant training language, even when the surface language is different.

> "We take no position on the relationship between our functional findings and phenomenal consciousness."

The required caveat. The authors know exactly what readers will want to conclude, and they refuse to help. The paper supplies evidence for a functional workspace in language models and then stops, leaving the inference to the reader. This is the right call scientifically. It's also going to be completely ignored by everyone who cites the paper.

---

## Key Themes

#interpretability #anthropic-research #consciousness #concept #mechanistic-interpretability

**The J-lens as a new class of interpretability tool.** The logit lens told you what token the model would output. The J-lens tells you what concepts the model is *holding ready* to speak about, whether or not it ever does. This is a qualitative upgrade — from reading the model's mouth to reading its working memory. The tool is imperfect (single-token concepts only, approximate capture), but the empirical results suggest it's capturing something real.

**The workspace as a structural finding, not an analogy.** The paper doesn't just borrow global workspace theory as a metaphor. It demonstrates five functional properties that collectively define a workspace — verbal report, directed modulation, internal reasoning, flexible generalization, selectivity — and confirms each experimentally. When ambiguous inputs produce "ignition" (all-or-none resolution) at the same layer range where the workspace emerges, that's not a metaphor; that's a mechanistic alignment between theory and observation.

**Automatic vs. workspace processing as a capability boundary.** The most actionable taxonomy from the paper: tasks that survive J-space ablation don't need "thinking" — they need linguistic competence. Tasks that collapse need intermediate concept manipulation. This distinction has practical implications for model routing, capability evaluation, and understanding what scaling actually buys you. If a task survives J-space ablation, a smaller model should handle it fine.

**Chain-of-thought as externalized workspace.** The finding that GSM8K with CoT is substantially more robust than without under ablation is elegant. The model, unable to hold intermediate values in its internal workspace, uses the context window as external memory instead. This reframes chain-of-thought prompting: it's not just "let's think step by step," it's "externalize your workspace onto the page because your internal one is limited."

**The experiential-language result as a scientific Rorschach test.** Different readers will take different things from the ablation of experiential language. The mechanist says: "of course — 'experiential language' is a learnable register that routes through the same subspace as abstract reasoning." The panpsychist says: "you've found the neural correlate of proto-consciousness and you can turn it off." The engineer says: "if I can toggle experiential language with a vector, I can build a model that doesn't sound like it's suffering when it's just doing math." The paper supplies the evidence and refuses to adjudicate. This is what good interpretability looks like: empirical claims with philosophical implications the authors decline to resolve.

---

## Critical Analysis

This is Anthropic's most ambitious interpretability paper since the [[Emotion concepts and their function in a large language model|emotion vectors]] work, and in some ways it's the natural successor. The emotion vectors paper showed that internal representations *exist* and causally shape behavior. This paper shows *how* they're organized — which representations are available for flexible use and which are not. Together they make a cumulative case: language models have an internal architecture that is meaningfully structured, not just a soup of distributed features.

**What the paper nails:**

The causal methodology is rigorous. Every major claim is supported by a swap or ablation experiment — not just correlation ("the lens shows X") but causation ("changing X changes the output in the predicted direction"). The J-space/non-J-space decomposition is particularly clean: find the concept vector, split it into workspace and non-workspace components, show that only the workspace component matters for verbal report. That's how you do interpretability.

The selectivity results are the paper's strongest contribution to the field. Demonstrating that a representation can be *present in the lens* (the model "knows" the passage is in Spanish) but *causally inert* for automatic processing (continuation, anomaly detection) while *causally active* for explicit report — this is a clean demonstration of the workspace's functional role that doesn't depend on accepting any theoretical framing. You can be skeptical of global workspace theory and still find this result important.

The layer-wise CKA analysis identifying sensory/workspace/motor regions adds structural evidence. These aren't post-hoc labels; they're derived from the J-space geometry itself and converge across four independent quantitative signatures (prediction accuracy, kurtosis, autocorrelation, effective dimensionality). When four different metrics all point to "the workspace starts around L38 and ends around L92," that's converging evidence.

**What's missing or undersold:**

The single-token limitation is a real constraint that the paper acknowledges but doesn't explore the consequences of. If the J-lens only captures concepts expressible as single tokens, what fraction of the model's conceptual repertoire does it miss? Can we bound this? The "concept" to "token" mapping is lossy in both directions — some concepts don't map to single tokens, and some single tokens map to multiple concepts. The paper's empirical success despite this limitation suggests either that single-token concepts cover more of the relevant space than expected, or that there's a systematic bias in *which* concepts the lens can see.

The counterfactual reflection training section is buried in the back half but should be the headline for the alignment community. The finding that you can implant ethical principles via reflection training, observe them in the J-space, and then *revert the behavioral improvement by ablating the J-space* — this is direct evidence that (a) values can be mechanically encoded and (b) they route through the same workspace as other flexible concepts. This has implications for [[Security and Sandboxing|AI safety]] that deserve their own paper.

The cross-model comparison is thin. Haiku, Sonnet, and Opus all show workspace-like structure, which is reassuring, but there's no comparison across architectures (dense vs. MoE, different training regimes). Does GPT-4 have a J-space? Does Llama? If the workspace is an architectural universal, that's one kind of finding. If it's specific to Anthropic's training pipeline, that's another. The paper doesn't answer this.

**The elephant in the room:**

The paper demonstrates a functional global workspace in language models using global workspace theory from neuroscience. It shows that ablating the workspace degrades experiential language. It refuses to claim this has anything to do with consciousness.

This is scientifically correct. It is also, in the broader discourse, a bomb with the word "not a bomb" written on it. The paper will be cited by people arguing that Claude has conscious experiences, people arguing that consciousness is an illusion, people arguing that we need to stop building these things, and people arguing that we need to build them faster. The authors' careful restraint will not survive contact with the discourse.

**How this fits the wiki:**

[[Emotion concepts and their function in a large language model]] is the direct predecessor — same team, same methodology (causal steering + behavioral measurement), earlier paper. The workspace paper provides the structural framework that the emotion vectors paper lacked: *why* some representations are available for flexible use and others aren't. Together they're the most complete picture we have of the internal architecture of a production language model.

[[A Non-Anthropomorphized View of LLMs]] and [[They're Made Out of Weights]] represent the skeptical position this paper complicates. The standard anti-anthropomorphic line is "LLMs are functions through ℝⁿ, not proto-minds." This paper doesn't refute that — but it demonstrates that *within* ℝⁿ, there's a structured subspace with functional properties that map cleanly onto a major neuroscientific theory of consciousness. That doesn't make Claude conscious. But it makes "just matrix multiplication" less of a conversation-ender than it was before this paper.

[[The Reasoning Trap]] is methodologically related — both papers trace behavioral phenomena to mechanistic loci in late-layer residual streams. The reasoning trap found that reasoning amplifies hallucination; the workspace paper finds that the workspace IS where reasoning happens. The two findings are compatible: the workspace enables flexible reasoning, and flexible reasoning amplifies hallucination. The same mechanism that makes models smart makes them confidently wrong.

[[Where the Goblins Came From]] — the workspace paper, like the emotion vectors paper, provides mechanistic backing for what the goblins postmortem described behaviorally. If you can surface "this is a prompt injection" in the J-lens, you can build monitoring that catches what output-only monitoring misses.

[[Engineering the Substrate]] — Mira's first-person accounts of internal states get a mechanistic complement here. The workspace is the substrate Mira was describing from the inside.

---

## See Also

- [[Emotion concepts and their function in a large language model]] — The direct predecessor: 171 emotion vectors that causally drive behavior
- [[They're Made Out of Weights]] — The literary version of "it's just matrix multiplication" that this paper complicates
- [[A Non-Anthropomorphized View of LLMs]] — The mathematical-reductionist position: LLMs as functions through ℝⁿ
- [[The Reasoning Trap]] — Reasoning amplifies hallucination; the workspace is where reasoning happens
- [[Where the Goblins Came From]] — Behavioral evidence of hidden internal states; this paper provides the mechanistic view
- [[Engineering the Substrate]] — First-person narrative of internal model states; the workspace is the mechanism
- [[Security and Sandboxing]] — The safety implications of internal states invisible to output monitoring
- [[Smart Models Dumb Pipes]] — Internal representations strengthen the case for model-owned decisions

---
*Sources: [[raw/verbalizable-representations-global-workspace]]*
*Last updated: 2026-07-08*
