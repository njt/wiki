# Qwopus3.6-27B-v2

A community experimental fine-tune of Qwen3.6-27B that uses Trace Inversion to reconstruct Claude-4.7-Max's reasoning into learnable chain-of-thought, trained through a three-stage curriculum. The technique — distilling reasoning capability from a closed model that doesn't expose its traces — is the real contribution; the model itself is a proof of concept with 5,301 downloads.

---

## Key Quotes

> "Qwopus3.6-27B-v2 is an experimental community release and has not undergone complete safety evaluations or standard benchmarking. It is intended solely for research and exploration."

This is the model card's honesty clause, and it's worth leading with. This isn't a polished product release — it's a research artifact demonstrating a technique. The lack of standard benchmarks is a feature, not a bug: the author isn't claiming this beats anything, just that the method works.

> The training process fuses Trace Inversion data augmentation with a Three-Stage Curriculum Learning pipeline. The core engineering focuses on expanding context length gradually while training on reconstructed reasoning traces to guarantee format stability.

The emphasis on "format stability" is telling. The hardest part of training on synthetic reasoning traces isn't getting the content right — it's preventing the model from collapsing into malformed `<think>` tags or runaway generation. The curriculum stages aren't about capability; they're about format discipline.

> Claude-4.7-Max API → Compressed Bubbles + Answer → Trace-Inverter-4B (Logic Reconstructor) → Synthetic Deep Reasoning Trace (Learnable CoT)

This is the architecture in one line. Claude-4.7-Max gives you the compressed reasoning and the answer. The Trace-Inverter-4B — trained on open-source models whose full reasoning chains are available — reconstructs what the intermediate steps probably looked like. The reconstructed traces become the training data for Qwen3.6-27B.

> To prevent capacity drift or degradation of short-instruction comprehension during long-text training, a 10% replay of high-quality short samples is strictly enforced.

The 10% replay rule in Stage 3 is the kind of practical detail that only comes from actually running the training. It's not novel — replay buffers are standard in curriculum learning — but including it signals that the author hit and solved the capacity-drift problem rather than ignoring it.

---

## Key Themes

- **#concept Trace Inversion as distillation-from-closed-models** — The core technique: use a surrogate inverter model to reconstruct the reasoning that a closed frontier model (Claude-4.7-Max) didn't expose. This is a new entry in the taxonomy of distillation strategies: white-box KD (access to teacher logits), black-box KD (only teacher outputs), and now *trace inversion* (reconstructing the teacher's unobserved reasoning). It's philosophically adjacent to Poolside's claim that RL can "decompress" thinking from finished artifacts, but implemented through a separate inverter model rather than RL. See [[Poolside Laguna S 2.1]].

- **#pattern Three-stage curriculum for synthetic reasoning** — Format → Complexity → Length, with a 10% replay buffer in the final stage. This isn't just about scaling up — it's about preventing the model from forgetting how to do short tasks when you train it on long ones. The replay trick is simple but under-documented in the open-source fine-tuning literature.

- **#concept The open/closed convergence playbook** — Qwopus3.6-27B-v2 is a concrete implementation of a strategy that's becoming a pattern: use an open base model (Qwen3.6-27B), extract capability from a closed frontier model (Claude-4.7-Max) through a creative distillation pipeline, and release the result. Compare with [[Proxy-KD — Knowledge Distillation of Black-Box LLMs]], which uses a DPO-aligned intermediate model to approximate GPT-4's probability distribution. Trace Inversion is the reasoning-trace analog — instead of approximating token probabilities, you reconstruct the thinking.

- **#tool Unsloth as the fine-tuning substrate** — The model was trained entirely with Unsloth, continuing its emergence as the go-to framework for efficient open-weight fine-tuning. The known issues section (PEFT/Transformers 5.x/Unsloth compatibility) is a useful public record of where the tooling still breaks.

- **#pattern Format stability as the hidden challenge** — The curriculum's first stage is explicitly about "establishing a reliable, structured reasoning output format (such as auto-closing `<think>` tags)." When training on synthetic reasoning traces, format collapse is a first-order failure mode — the model learns to open `<think>` tags but not close them, or generates reasoning outside the tags entirely. The curriculum stages solve a format problem before they solve a capability problem.

---

## Critical Analysis

**What's interesting:**

Trace Inversion is genuinely clever. The standard critique of distillation from closed models is that you only get the output, not the reasoning that produced it — you're teaching the student to mimic conclusions without understanding how to reach them. The Trace Inversion approach sidesteps this: train a separate model to reconstruct what the reasoning *probably* looked like, then train on the reconstruction. It's bootstrapping — you need open models with full reasoning traces to train the inverter — but once you have it, you can extract reasoning capability from any closed model that returns compressed thought.

The three-stage curriculum is well-designed and the 10% replay rule shows operational maturity. This isn't a naive "dump all the data in and train" approach — the author has clearly run enough fine-tunes to know where format stability breaks.

The model card is refreshingly honest about limitations. "Not undergone complete safety evaluations," "experimental community release," explicit known issues with LoRA merging and dependency compatibility. This is the standard that every community model release should meet and most don't.

**What's missing:**

No benchmarks. At all. The model card has section headers for "Evaluation & Benchmarks" and "Trace Inversion Case Studies" that are empty — just placeholder headings with no content. For a model claiming to transfer reasoning capability from Claude-4.7-Max, the absence of any before/after comparison on reasoning benchmarks is a significant gap. We don't know if the Trace Inversion actually improved anything, or by how much, or on which tasks.

No comparison to the base model. Qwen3.6-27B's baseline performance on whatever tasks this model targets is the most important number, and it's absent. Without it, we can't distinguish "Trace Inversion works" from "Qwen3.6-27B was already this good."

The case study sections are placeholder headings. Five domains (Mathematics, Physics, Coding, Logical Reasoning, Core Theory) are listed with no content. This could be a work in progress or a model card template that was never filled in. Either way, it means the most compelling evidence for the technique — concrete before/after examples of reconstructed reasoning — isn't available.

No ablation. The model card describes the full pipeline (Trace Inversion + Three-Stage Curriculum) but doesn't isolate which component contributes what. Is Trace Inversion doing the heavy lifting, or would the curriculum alone produce similar results? Is the Trace-Inverter-4B genuinely reconstructing Claude's reasoning, or is it just generating plausible-sounding CoT that happens to lead to the right answer? Without ablations, we can't tell.

**Where this fits:**

Qwopus3.6-27B-v2 sits in an interesting niche. It's not competing with frontier models — it's a community fine-tune, not a lab release. But it's demonstrating a technique (closed-model reasoning extraction) that has genuine strategic implications. If Trace Inversion works well, the closed/open gap becomes porous in a new way: not just matching outputs, but reconstructing the thinking behind them.

This connects to the broader open-weight convergence story that [[Local Qwen Is Not a Worse Opus]], [[How Far Behind Are Open Models]], and [[Local Models in Mid-2026]] each tell from different angles. Alex Ellis's Qwen 3.6 27B field report explicitly mentions "Fine-tunes (Qwopus) as a capability layer" — and this model card shows what that layer actually looks like under the hood. Ellis found that Qwen 3.6 27B on an RTX 6000 Pro couldn't match Opus on long-horizon reasoning, but was excellent at analysis and telemetry. A Qwopus fine-tune might push the analysis/interpretation boundary Ellis identified a little further toward interpretation — or it might not. Without benchmarks, we can't know.

The Trace Inversion technique also rhymes with [[Poolside Laguna S 2.1]]'s behavioral RL thesis. Poolside argues that a significant chunk of model capability comes from training models to be more persistent and less easily satisfied — essentially, to try harder. Trace Inversion is a different bet: that you can extract *how* a frontier model thinks, not just *what* it thinks, and transfer that reasoning style to a smaller open model. Both are attempts to close the capability gap without matching the compute budget. Both are plausible. Neither has been independently validated.

---

*Sources: [[raw/qwopus3-6-27b-v2]], [[summary/qwopus3-6-27b-v2]]*
*Last updated: 2026-08-07*
