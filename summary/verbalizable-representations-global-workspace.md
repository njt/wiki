---
url: https://transformer-circuits.pub/2026/workspace/index.html
title: "Verbalizable Representations Form a Global Workspace in Language Models"
author: "Wes Gurnee*, Nicholas Sofroniew*, Adam Pearce, Mateusz Piotrowski, Isaac Kauvar, Runjin Chen, Anna Soligo, Paul Bogdan, Euan Ong, Rowan Wang, Ben Thompson, David Abrahams, Subhash Kantamneni, Emmanuel Ameisen, Joshua Batson, Jack Lindsey*†"
date_fetched: 2026-07-08
date_published: 2026-07-06
topics:
  - ai-research-and-models
---

Anthropic researchers present evidence that language models have a privileged set of internal representations — a "global workspace" — available for verbal report, modulation, and flexible reasoning, sitting atop a much larger volume of automatic processing.

The key tool is the **Jacobian Lens (J-lens)**, which computes the average linearized effect of each layer's activations on the model's likelihood of producing a given token, averaged over many contexts. This distinguishes representations the model is *poised* to verbalize from those it merely happens to verbalize in one context. The collective set of J-lens vectors forms the **J-space**, a sparse subframe within the activation space accounting for less than 10% of total variance.

The paper demonstrates five functional properties of a workspace:

**Verbal report**: When asked what it's thinking, the model names concepts present in the J-space. Swapping J-space vectors changes its answer; swapping non-J-space components (the other ~93% of variance) has almost no effect.

**Directed modulation**: The model can hold concepts in mind or perform silent mental arithmetic while doing unrelated tasks. Under "ignore" instructions, target concepts still appear in the J-space at reduced but non-zero rates — a parallel to the human "white bear effect."

**Internal reasoning**: The J-lens surfaces intermediate computations before the answer is reached. In two-hop reasoning ("legs on the animal that spins webs"), "spider" appears in intermediate layers though never in the prompt or output. Multi-step arithmetic reveals intermediate values appearing in computational order across successive layers. Swapping intermediates redirects the model's conclusion.

**Flexible generalization**: The same J-space swap (e.g., France → China) affects multiple downstream computations (capital, language, continent) consistently, demonstrating broadcast generality.

**Selectivity**: The J-space is causally involved in abstract reasoning but not in routine processing like grammatical fluency, anomaly detection, or line-wrap continuation. Ablating the J-space severely degrades multi-hop reasoning, analogy, translation, and summarization while leaving MMLU multiple-choice, sentiment classification, and extractive QA largely intact.

Structurally, the J-space organizes into three layer regions: sensory (early), workspace (middle, ~L38–L92), and motor (late). At the workspace onset, activations show "ignition" — an all-or-none amplification of one interpretation from ambiguous inputs.

The paper explores applications including prompt-injection detection, counterfactual reflection training (implanting ethical principles that show up in J-space and are reversed by ablation), and analysis of post-training changes. Under J-space ablation, the model's experiential language collapses — narration becomes detached and mechanical, affecting descriptions of both its own processing and other people's experiences.

The authors take no position on phenomenal consciousness and note several differences from biological global workspace theory, including the absence of recurrent loops (broadcast occurs within a single feedforward pass).
