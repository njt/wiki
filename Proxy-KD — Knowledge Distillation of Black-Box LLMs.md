# Proxy-KD — Knowledge Distillation of Black-Box LLMs

A 2024 paper from Sun Yat-sen University and Alibaba that solves a practical problem: how do you distill GPT-4's capabilities into a 7B model when you can only see GPT-4's outputs, not its internal probability distributions? The answer — insert an aligned proxy model in between — is clever, but the paper's real contribution is proving that black-box teacher distributions are *worth approximating*, even at significant computational cost.

---

## Key Quotes

> "Proxy-KD not only enhances the performance of KD from black-box teacher models but also surpasses traditional white-box KD techniques."

The headline claim. Proxy-KD with a Llama-2-7B student achieves **56.78% average** across six benchmarks vs. 53.66% for vanilla black-box KD and 51.27% for white-box KD. The proxy architecture beats having *actual access* to the teacher's internals — because GPT-4 as a black box is better than Llama-2-70B as a white box.

> "The capabilities of closed-source teachers are more beneficial than those of open-source teachers, even after alignment."

This is the finding that should make open-source advocates uncomfortable. You can align a 70B open model to GPT-4's outputs, then use that aligned model as a white-box teacher — and the student *still* does worse than Proxy-KD. The gap between closed and open isn't just about output quality; it's about the *structure* of the probability distribution, and Proxy-KD captures that structure better than direct alignment.

> "Preference optimization is crucial for refining the proxy model's ability to emulate the teacher effectively."

DPO alignment accounts for most of Proxy-KD's gains. Removing it drops BBH by 10.40 points and GSM8K by 5.53. The DPO loop runs for k=16 iterations, each taking 28 hours on 8×A100s for the 70B proxy. This is not a cheap technique.

> "TAKD's failure to account for the proxy alignment process, which is essential for effective closed-source KD."

A polite way of saying the prior work (Teacher Assistant KD from 2020) got the right idea but missed the hard part. Just sticking an intermediate model in the pipeline doesn't help if that model isn't aligned to the teacher's distribution. The alignment phase *is* the technique; the proxy is just the vehicle.

---

## Key Themes

- **#concept Black-box knowledge distillation** — The defining constraint: you have API access to GPT-4 but no logits, no hidden states, no attention maps. Just input-output pairs. This is the actual situation for anyone building on top of proprietary LLMs, and it will remain the situation as long as frontier models are gated behind APIs.

- **#pattern Proxy-mediated transfer** — Insert a trainable proxy between teacher and student. The proxy is larger than the student (70B vs. 7B) so it can absorb the teacher's distribution, then the student distills from the proxy's *distribution* rather than just the teacher's outputs. This is a general pattern worth stealing for any transfer-learning problem where the source is opaque.

- **#pattern Sample-level confidence weighting** — w(x,y) = σ[(log π_p(y|x) − μ) / γ]. When the proxy's probability for the teacher's answer is high, the student trusts that sample more. When the proxy is uncertain, the student downweights it. Simple, obvious in retrospect, and demonstrably effective. This should be a default component in any distillation pipeline.

- **#tool DPO (Direct Preference Optimization)** — The alignment workhorse. For each input, the teacher's response and the proxy's current response form a preference pair. DPO iteratively pushes the proxy toward the teacher's preferences. The 16-iteration loop is expensive but the gains are real.

- **#pattern Capacity gap as a first-class problem** — The reason you need a proxy at all is that the teacher (GPT-4) and student (Llama-2-7B) are too far apart. Direct distillation fails because the student literally cannot represent the teacher's distribution. The proxy bridges the gap by being closer to the teacher's scale. This is the "model capacity gap" problem (Cho and Hariharan, 2019) applied to LLMs — and it's going to matter more as frontier models pull further ahead.

---

## Critical Analysis

**The proxy is a crutch, not a solution.** Proxy-KD works — the numbers are convincing — but the architecture is fundamentally wasteful. You're training a 70B model (the proxy) as an *intermediary*, not a deliverable. The student (7B) is what you keep; the proxy is discarded. If you have the compute to train a 70B proxy, you could instead train the 7B student on more data with simpler methods and possibly get similar results. The paper doesn't compare against "just use 4× more training data with vanilla KD," which is the obvious baseline for anyone with the GPU budget Proxy-KD requires.

**The model family monoculture is a real limitation.** The authors acknowledge they only tested Llama (Llama-2-70B proxy, Llama-1/2-7B students). No Qwen, no Mistral, no Gemma. Knowledge distillation is sensitive to architecture similarity — teacher and student with different tokenizers or attention patterns might show different gains. The claim "surpasses traditional white-box KD techniques" is true on this setup but may not generalize to other model families. This isn't a fatal flaw, but it limits how confidently you can apply Proxy-KD to, say, distilling Claude into Gemma.

**The DPO cost makes this research, not production.** 28 hours per round × 16 rounds = 448 GPU-hours just for proxy alignment, plus the actual student training. On 8×A100s at ~$2/GPU-hour, that's roughly $7,000 in compute for one distillation run. For a 7B model that you can now get better distillation results from simpler techniques (see [[Self-Distillation]]), that's hard to justify. The paper's contribution is conceptual — proxy-mediated transfer works — not practical. Someone needs to make it cheaper.

**The "beats white-box KD" result is the paper's most important and most fragile claim.** The white-box baseline uses Llama-2-70B-Chat as the teacher — which is itself a fine-tuned model that may not have the same distribution quality as a base model. A fairer comparison would use GPT-4's outputs to train a white-box teacher, then distill from that. The paper does this (aligned proxy as white-box teacher) and Proxy-KD still wins, but the margin shrinks. This suggests the proxy's value is partly in the *process* (DPO alignment + weighted KL) and partly in having a better teacher distribution to approximate.

**What's genuinely new here.** The three-component design (proxy alignment via DPO + weighted KL distillation + sample-level confidence weights) is modular and each piece has independent value. You could take just the sample-level weighting and apply it to any distillation pipeline. You could take the DPO alignment loop and use it to improve any model's emulation of a black-box teacher. The paper is a menu of techniques, not just one technique — and the best papers work that way.

**What this means for the open-source frontier.** The paper was written in early 2024, before DeepSeek R1 narrowed the open/closed gap (see [[How Far Behind Are Open Models]]). In a world where the gap is 8-10 months and widening, Proxy-KD becomes *more* valuable, not less. If you can't build GPT-5, you can at least steal its probability distribution through a proxy. The technique is a form of model espionage — extracting structural knowledge from outputs alone — and it will get more attention as frontier models become more capable and more closed.

---

## See Also

- [[Self-Distillation]] — The other end of the distillation spectrum: the model improves using *only* its own outputs, no teacher at all. SSD and Proxy-KD are complementary — one works when you have no teacher, the other when your teacher is too good to imitate directly.
- [[Granite Libraries and Project Granite Switch]] — IBM's adapter-function approach to modular model expertise. Proxy-KD's weighted KL objective is a form of selective knowledge transfer; Granite's aLoRA switching is a form of selective knowledge activation. Different mechanism, same goal: don't make the model learn everything at once.
- [[Fine-Tuning a Local LLM to Categorize Questions]] — The pragmatic end of the distillation spectrum: a 600M model fine-tuned for a single task. Proxy-KD targets general capability transfer; Helgevold targets narrow reliability. The opaque output encoding trick (two-char IDs instead of semantic labels) is conceptually similar to the weighted KL mechanism — both are about controlling *which* signals the student attends to.
- [[How Far Behind Are Open Models]] — Ihle's quantification of the open/closed gap. Proxy-KD is one answer to "what do we do about the gap?" — if you can't match frontier models, distill them.
- [[Data Engineering for Large Models]] — The OpenOrca and Nectar datasets used in Proxy-KD are examples of the data-centric approach this textbook covers. Proxy-KD's performance depends on the quality of the teacher's outputs; garbage teacher outputs → garbage proxy alignment → garbage student.
- [[Recent Developments in LLM Architectures]] — Raschka's survey of long-context cost reduction techniques. Proxy-KD's 1024-token max sequence length is a relic of early-2024 constraints; the technique would benefit from the cheaper long-context inference these architectures enable.

---
*Sources: [[raw/2401.07013v2]]*
*Last updated: 2026-07-03*
