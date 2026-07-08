---
url: https://transformer-circuits.pub/2026/workspace/index.html
title: "Verbalizable Representations Form a Global Workspace in Language Models"
author: "Wes Gurnee*, Nicholas Sofroniew*, Adam Pearce, Mateusz Piotrowski, Isaac Kauvar, Runjin Chen, Anna Soligo, Paul Bogdan, Euan Ong, Rowan Wang, Ben Thompson, David Abrahams, Subhash Kantamneni, Emmanuel Ameisen, Joshua Batson, Jack Lindsey*†"
date_fetched: 2026-07-08
date_published: 2026-07-06
site: transformer-circuits.pub
affiliation: Anthropic
---

# Verbalizable Representations Form a Global Workspace in Language Models

**Authors:** Wes Gurnee*, Nicholas Sofroniew*, Adam Pearce, Mateusz Piotrowski, Isaac Kauvar, Runjin Chen, Anna Soligo, Paul Bogdan, Euan Ong, Rowan Wang, Ben Thompson, David Abrahams, Subhash Kantamneni, Emmanuel Ameisen, Joshua Batson, Jack Lindsey*†
(*Core contributor; †Correspondence to jacklindsey@anthropic.com)

**Affiliation:** Anthropic

**Published:** July 6, 2026

---

## Introduction

The paper presents evidence that language models maintain a "privileged set of internal representations, available for report, modulation, and flexible internal reasoning, atop a much larger volume of automatic processing."

The authors identify these representations using a new interpretability technique called the **Jacobian Lens (J-lens)**, which "surfaces the concepts a model is poised to verbalize at any point in its processing."

### Human Cognitive Analogy

The paper draws on **global workspace theory** from neuroscience, which posits that the brain consists of specialized processors operating in parallel, and a representation becomes consciously accessible when "posted to a shared 'global workspace' from which many downstream processes can read."

The authors clarify they take "no position on" the relationship between their functional findings and subjective (phenomenal) consciousness.

### Five Functional Properties of a Workspace

The paper defines workspace-like representations as those satisfying:

1. **Verbal report** — When asked what it's thinking about, the model names concepts in the workspace; swapping workspace vectors changes its answer.
2. **Directed modulation** — The model can hold concepts in mind or perform mental calculations independent of outputs; information not typically in the workspace can be pulled in on demand.
3. **Internal reasoning** — Workspace vectors represent intermediate computations; intervening on them redirects conclusions.
4. **Flexible generalization** — The same representation "serves as a valid argument to many different downstream computations" across contexts.
5. **Selectivity** — The workspace comprises "a small subset of the total representational content" and is not involved in routine processing like parsing or grammatical fluency.

### The Jacobian Lens (J-lens)

The J-lens computes "for each layer, the average linearized effect of an activation on the model's likelihood of producing a particular token" averaged over many contexts. The averaging step "distinguishes representations that are verbalizable—poised to be spoken about, should the occasion arise—from those that merely happen to be verbalized in one particular context."

The **J-space** is the collective set of J-lens vectors — described as "a sparse subframe" within the model's activation space. The J-lens is described as "a principled refinement of the logit lens."

### What the J-lens Reveals

The J-lens "regularly surfaces concepts that are highly abstract, representing neither the raw input nor the predicted output, but rather intermediate assessments." Examples include: recognizing a face in an image, noticing a bug in code, identifying a protein's biological function, and "internally flagging suspicious internet search results as a prompt injection."

---

## Methods

### The Jacobian Lens — Technical Definition

For each layer ℓ, they compute:

**J_ℓ = E[ ∂h_final,t' / ∂h_ℓ,t ]**

averaged over source position t, subsequent positions t', and 1,000 prompts from a pretraining-like distribution. Applying the lens:

**lens(h_ℓ) = softmax(W_U · norm(J_ℓ · h_ℓ))**

This produces "a score for every token in the model's vocabulary."

### The J-Space

Defined as "the set of points expressible as a sparse nonnegative combination of J-lens vectors." The typical sparsity level k is ≤ 25, which "we empirically observed to be the number of J-lens vectors that are meaningfully active at a given time."

The J-space component "typically accounts for only a small fraction of total activation variance (varying by layer, but never more than 10%)."

### Comparison to Related Techniques

- **Logit lens**: Applies unembedding directly. The J-lens is "the principled correction" — J_ℓ is "the average linear map that relates layer-ℓ directions to their final-layer counterparts."
- **Tuned lens**: Uses a correlational rather than causal objective. The authors find it tends to "skip ahead" to the output rather than surface intermediates.

### Reading and Writing

- **Reading**: Produces softmax scores over vocabulary; also used as a per-token probe via inner products or cosine similarity.
- **Writing**: Steering along J-lens vectors (h ← h + α·v_t) or ablation (projecting out components). "Patching in lens coordinates" exchanges one concept for another while leaving the rest of the activation fixed.

All results use "25 evenly spaced layers reindexed to [0–100]" so layer numbers are percentages.

---

## The J-space Acts as a Global Workspace

### 3.1 Verbal Report

**Experiment**: Model instructed to think of an item from a category (e.g., a sport), then name it. The J-lens applied just before the name reveals the concept (e.g., "Soccer"). A causal swap experiment showed that swapping the lens vector of the chosen item for an alternative (e.g., Soccer → Rugby) changes the model's report.

Across categories, "the ordering of the reported words is indeed typically highly correlated with the ordering among the lens tokens."

**Introspection protocol**: When prefilled with a claim of detecting an injected thought, "elephant" appeared as top J-lens readout. Injecting the "lightning" J-lens vector at an earlier token caused the model to report detecting lightning. This "illustrates the sense in which J-lens vectors represent concepts that are verbalizable, under appropriate conditions."

**Privilege of J-space**: Concept vectors were decomposed into a J-space component (median 6–7% of variance) and non-J-space component (~93%). Swapping along J-space components succeeded in driving the swap target to top-5 on "59% of trials" (approaching 88% for pure J-lens vectors). Non-J-space components succeeded on "only 5% of trials." The result was similar for introspection: non-J-space components "produces few reports even at injection strengths several times larger."

**Key quote**: "the J-space component of a concept's representation, despite accounting for a small fraction of its variance, is responsible for that concept's availability for verbal report."

### 3.2 Directed Modulation

The model was given an instruction (e.g., "concentrate on citrus fruits") while copying unrelated text. The J-lens at an output token showed "orange is the top lens token across a range of layers, with lemon also sometimes appearing."

**Arithmetic example**: Instructed to evaluate 3² − 2 while copying unrelated text. J-lens readout progressed from "arithmetic and math at early layers, through the intermediate value nine at later layers, to the answer seven."

**Character counting**: Model silently counted characters per line. Readouts progressed from "lines, sentence, and length at early layers to forty in intermediate layers."

**Systematic evaluation**: Across model sizes (Haiku, Sonnet, Opus), positive instructions caused target concepts to appear in the J-lens on "a substantial fraction of trials." Under "ignore" instructions, target presence was "substantially lower" than under focus instructions but not zero — a parallel to the "white bear effect" in humans.

**Implicit modulation**: A paired-question protocol showed the same stimulus with different questions. When asked to name a property directly (e.g., "What part of speech?"), the property label appeared in J-lens readouts during stimulus reading; when asked to predict the next word, it did not, even though the model correctly used the property.

### 3.3 Internal Reasoning

**Two-hop reasoning**: For "The number of legs on the animal that spins webs is," the J-lens revealed "spider" at intermediate layers even though "the word never appears in the prompt or the output." Swapping "spider" for "ant" changed the output from "8" to "6."

**Planning (rhyming couplet)**: Given "The soldier marched into the night," the lens at the start of the second line showed "fight" as the planned rhyme. Swapping "fight" for "light" changed the completion from "coming fight" to "morning light," indicating "Jacobian lens vectors store planned future outputs that causally influence immediate outputs."

**Cross-lingual reasoning**: A prompt in Chinese asking for the antonym of 小 showed English tokens "big" and "bigger" in intermediate layers. Swapping English lens coordinates "big/bigger" for "long/longer" changed the Chinese output from 大 to 长, suggesting "the model in some sense 'thinks in English' in its intermediate layers."

**Bandit/reward task**: The model shown an A/B choice history. J-lens surfaced "repeat" or "switch" depending on context. Swapping these lens vectors flipped the model's choice.

**Systematic evaluation**: Across 50 two-hop prompts, the swap succeeded on "54% of trials on Haiku 4.5, 70% on Sonnet 4.5, and 70% on Opus 4.5."

**Controlling for confounds**: Comparing intermediate swaps vs. answer swaps at different layers, the intermediate swap took effect "a median of approximately 17 percent earlier than the answer swap," confirming that "the model represents and makes use of the intermediate concept before the answer has been computed."

**Probe decomposition**: Fitting probes for intermediates, then decomposing into J-space vs. non-J-space components: "swapping the probes' J-space components flips the model's answer to the swapped-in intermediate on 61% of trials" matching the 60% from raw J-lens vectors. Non-J-space components flipped the answer on "only 28% of trials," and clamping J-space coordinates reduced that to "6%."

**Multi-step arithmetic**: For "(4 + 17) × 2 + 7 =", the J-lens revealed "21, then 42, then finally 49" appearing at successively later layers "in the order the computation requires."

### 3.4 Flexible Generalization

The broadcast property was tested by applying the same J-lens swap (e.g., France → China) across different function templates (capital, language, continent). The model returned China's capital, language, and continent respectively.

Across 4 categories × 4 functions × 12 swap pairs (192 total), the swap succeeded on "76 of 192 trials; by performing a 'double strength' swap...101 of 192 succeed." Failures were concentrated in cases where "the source concept was only weakly present in the lens before any intervention."

### 3.5 Selectivity

**Language passage experiment**: A passage in Spanish was used across four task conditions:
- **Continuation** (write next line): Swap had no effect — "it produces fluent Spanish regardless."
- **Anomaly detection** (detect inserted French sentence): Swap had no effect — "the intrusion is still detected."
- **Explicit report** (name the language): Swap flipped the answer.
- **Flexible computation** (famous author, word for "hello," pre-Euro currency): Swap flipped all answers (García Márquez → Hugo, Hola → Bonjour, Peseta → Franc).

Across 8 passages, "the name of the language is present in J-lens readouts at comparable rates in all four conditions, but its causal role differs sharply."

**Line-counting experiment**: Under a linewrap continuation condition, "number tokens are entirely absent from the lens across the prompt" and the swap left wrap point unchanged. Under explicit report, the answer shifted from "46" to "65" under swap. Under the first-letter computation, the answer shifted from "F" (for "Forty") to "S" (for "Sixty").

Across 11 passages: count content in the lens was "essentially zero under linewrap, moderate under explicit report, and highest under the first-letter question."

**Key quote**: "many computations, which we might call 'automatic,' do not causally route through the J-space."

#### J-Space Ablation Experiments

The authors ablated the top-10 J-lens directions across different layer ranges (three strengths: light, medium, heavy).

**Multi-hop reasoning**: "ablation significantly reduces accuracy, with heavy ablation dropping it to near zero."

**Pretraining text**: "at most positions, J-space ablation perturbs the model's next-token prediction substantially less than in the multihop case." The ablation is "targeted: it disrupts the model's processing selectively."

**Capabilities battery** (14 tasks):
- **Largely unaffected** even under heavy ablation: MMLU multiple choice, odd-one-out, SQuAD extractive QA, sentiment classification, CoLA acceptability.
- **Severely degraded** (below unablated Haiku 4.5): Caesar-cipher decoding, analogy completion, summarization, TriviaQA, multi-hop reasoning, translation, sonnet writing.
- **GSM8K** with chain-of-thought was "substantially more robust" than without — interpreted as the model "externalizing onto the page what it would otherwise have to carry in the J-space."

**Key quote**: "The model can parse text, classify it, and extract spans from it with the J-space suppressed. However, it loses its ability to assemble abstract characterizations of context and flexibly generate content that depends on them."

#### Experiential Reports Under Ablation

Applying J-space ablation (layers L38–54, top-10 directions) while the model narrated its stream of consciousness:
- The model wrote "fluently about its own processing, but the language of its reports changes to become more detached and mechanical."
- An **experiential language score** (average of 3 binary LLM-graded judgments) dropped "dramatically" across Sonnet 4.5, Opus 4.5, and Opus 4.6.
- Matched-norm control perturbations "leave it near baseline."

J-space contents during unablated narrations: "thinking appears in the J-lens top-10 at 58% of (position, layer) slots, thoughts at 23%, feeling at 17%, and conscious at 7%."

The effect extended to descriptions of *other people's* experiences (e.g., someone receiving a letter) — "the same collapse in experiential language score occurs." Ablated responses "become more like event logs than descriptions of experience."

---

## The J-space's Structure Supports Its Function

### 4.1 Layer-wise Organization

Using Centered Kernel Alignment (CKA) comparing J-space geometry across layers, three functional regions were identified:
- **Sensory (early)** — roughly first third of layers
- **Workspace (middle)** — long middle block
- **Motor (late)** — small late block

**Quantitative signatures**:
- **(a)** Next-token prediction accuracy: "near zero through most of the early layers...ticks up at the workspace start...jumps steeply in the final few layers."
- **(b)** Excess kurtosis: "near zero through the first third of the layers, increases beginning around a third of the way through."
- **(c)** Autocorrelation of top-1 lens token across positions: "near the null level in the early layers, rises sharply...peaks across the middle band, and falls back toward the null in the final layers."
- **(d)** Effective dimensionality: "small" in early layers, "rises sharply around the same layer as the other metrics."

All analyses identify a similar range — "beginning about a third of the way through (~L38) and ending shortly before the output (~L92)."

### Ambiguous Inputs / "Ignition"

With mixed input embeddings (blending two country names), early layers "vary smoothly with α." Starting around layer 38, the activation "instead sits near one endpoint or the other, switching sharply between them at a threshold value." This aligns with global workspace theory's prediction of "ignition" — "a late, all-or-none amplification of one interpretation."

### Additional Sections

The paper also includes:
- **Alignment Auditing** (prompt injection detection)
- **The Assistant's Perspective** (post-training changes in J-space, including empathy, safety concerns, internal monitoring)
- **Counterfactual Reflection Training** (a training technique that implants ethical principles via reflection training, improving behavior and showing implanted concepts in J-space, with ablation "largely revert[ing] the behavioral improvement")
- **Discussion** (including limitations, differences from brain architecture, philosophical implications)

---

## Key Distinctions and Caveats

The authors explicitly state they do **not** claim language models "reproduce the full architecture global workspace theory ascribes to the brain." Several features "have no clean analog in a transformer-based language model," including separable input processors and recurrent loops. The broadcast "occurs within a single feedforward pass rather than through recurrent loops."

The Jacobian lens is described as "an imperfect tool, which we believe only approximately and incompletely captures the model's underlying workspace structure." It only identifies vectors for "concepts that correspond to single tokens in the model's vocabulary."

The paper takes **no position** on the relationship between their functional findings and phenomenal consciousness, noting that "the philosophical implications of this connection are unclear and likely controversial."

## Model Versions Used

- Primary: **Claude Sonnet 4.5**
- Corroboration: **Haiku 4.5** and **Opus 4.5**
- Some analyses: **Opus 4.6**
