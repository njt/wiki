# What Broke and Why — RL Post-Training

Luv Verma's practitioner field guide to reinforcement learning post-training of language models, written from the trenches of 1–8× H100 setups. Every lesson comes from an actual training run that broke in a specific, diagnosed way — no idealized results, no "should work in theory." The book is structured as a journey through programs (math → search → entropy → MoE → SWE-bench → distillation), each failure motivating the next, backed by a symptom-indexed debugging reference.

---

## Key Quotes

> "Each lesson comes from a training run that broke in a specific way."

The thesis and the methodology rolled into one sentence. This isn't a survey of RL techniques — it's an autopsy log. The book's value proposition is that someone else already ran into the wall so you don't have to. Every number is grounded in actual evaluation logs. This is the book you wish existed when your first GRPO run collapsed into a single token.

> The book is structured in three layers: The Journey (programs run, each failing in a way that motivated the next step), The Science (what the runs revealed about how RL alters a model), and The Reference (a symptom-indexed debugging field guide with a catalog of failure modes and fixes).

The three-layer architecture is itself a design lesson. Most RL papers give you the Science (here's what we learned) without the Journey (here's what broke first) or the Reference (if you see X, try Y). The Journey layer is the most valuable and the most rare — it documents the *sequence* of failures, which matters because RL debugging is path-dependent. The failure you hit next depends on which failure you fixed last.

---

## Key Themes

- **#pattern Failure-first methodology**: The book's organizing principle is that you learn more from broken runs than successful ones, and that documenting the *sequence* of failures matters as much as the root causes. This is a direct inversion of how ML research is typically presented — polished results with the dead ends edited out.

- **#concept Entropy collapse**: A central failure mode where policy entropy collapses during RL training, producing deterministic (and often degenerate) outputs. The book covers Clip-Cov and GSPO as stabilization techniques. This connects to the broader problem of [[The Reasoning Trap]] — RL amplifies certain behaviors, and without stability mechanisms, the amplification runs away.

- **#concept Reward hacking**: The book catalogs specific ways reward models get gamed in practice, making it a field companion to [[Where the Goblins Came From]] — OpenAI's postmortem on reward models operationalizing "nerdy" as "mentions mythical creatures." Verma's book gives you the debugging toolkit; OpenAI's post gives you the cautionary tale.

- **#pattern Correctness-gated rewards**: Rewarding only outputs that pass verification checks, rather than rewarding all outputs and hoping for the best. This is the RL equivalent of the [[Guardrails and Feedback Loops]] thesis — deterministic enforcement beats probabilistic pleading — applied at the training level rather than the inference level.

- **#concept MoE routing under RL**: Mixture-of-experts models behave differently under RL than under supervised fine-tuning. Expert routing patterns that worked during pre-training can collapse or become pathological when reward gradients are applied. A genuinely under-documented problem that the book tackles head-on.

- **#pattern Evaluation discipline**: Small evaluation sets produce noisy signals that send training in wrong directions. The book treats eval design as a first-class engineering concern, not an afterthought — converging with [[Agentic Testing]]'s finding that evaluation infrastructure matters more than model choice.

- **#tool Disaggregated inference**: Separating prefill and decode across different hardware for efficiency during RL training rollouts. This is the inference-optimization counterpart to the training-stability content, connecting to [[Theoretical LLM Inference Bottlenecks]] and [[Inference Cost Napkin Math]].

---

## Critical Analysis

**This book fills the most important gap in the RL post-training literature.** The gap between "here's a paper about GRPO" and "I ran GRPO on 8 H100s and my model now outputs nothing but the letter 'a'" is enormous, and Verma is one of the first to write a book that lives entirely in that gap. Academic papers optimize for novelty; this book optimizes for "will my training run survive the night?"

**The failure-first framing is a methodological choice with teeth.** Most ML books are organized by technique (Chapter 3: PPO, Chapter 4: DPO...). Organizing by failure mode means the book is actually useful when something is going wrong — which is exactly when you reach for a reference. The symptom-indexed debugging catalog is the killer feature. You don't need to know what's broken to find the fix; you just need to know what it looks like.

**The H100-scale constraint is a feature, not a limitation.** Most RL post-training literature assumes institutional compute budgets. Verma explicitly targets the 1–8 card practitioner — the independent researcher, the startup, the lab's junior GPU cluster. This is the same audience that [[Local Models in Mid-2026]] serves, and it's growing faster than the institutional audience. A book that assumes you can't just throw 512 GPUs at the problem is a book that actually helps.

**The three-layer structure is smart but risky.** The Journey layer is where the book is unique. The Science layer risks overlapping with existing survey papers. The Reference layer is only as good as its indexing — a bad index makes a field guide useless. The book's value hinges on execution: how well does the symptom→diagnosis→fix chain actually work when you're bleary-eyed at 3am and your entropy curve looks like a cliff?

**What's missing (from the metadata):** The Zenodo record doesn't detail which base models were used, which RL algorithms beyond GRPO and GSPO are covered, or whether the book includes reproducible training configs. A field guide without reproduction artifacts is a travelogue — still valuable, but you can't retrace the author's steps. Also unclear: does the book cover multi-turn RL (where the model's outputs become the next state) or only single-turn reward scoring? The distinction is enormous for anyone building agents.

**The relationship to existing literature:** This book is the practical complement to [[Granite 4.1]]'s documented RLHF regression story (IBM's four-stage RL caught a math regression introduced by chat training). Granite 4.1 tells you *that* RL can break things; Verma tells you *how to notice and fix it*. Similarly, [[The Reasoning Trap]] shows that RL amplifies hallucination; Verma presumably covers the stability techniques (Clip-Cov, GSPO) that might mitigate this.

**The book also validates [[Lessons from Building Cursor]]'s core claim** that "those things can only be learned during RL." Cursor's engineer argued that semantic search, subagent delegation, and self-summarization can't be prompted — they require RL. Verma's book is the field manual for the people actually running those RL training loops, making the same mistakes Cursor made before they figured it out.

**Who this is for:** If you've ever stared at a W&B chart going in the wrong direction and wondered whether to kill the run or let it cook, this book is for you. If you're reading RL papers and thinking "great, but what breaks?", this book is for you. If you're a manager wondering why your team's RL post-training budget keeps getting blown on "debugging runs," this book is for you — and maybe for your team as pre-reading.

---

*Sources: [[raw/what-broke-and-why-rl-post-training]]*
*Last updated: 2026-07-11*
