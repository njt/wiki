# Context Is Not Learning

Aravind Jayendran's sharp distillation of why the prevailing instinct to solve LLM learning through longer context windows mistakes one necessary memory system for the only one. Context and weights both shape activations, but through fundamentally different mechanisms — context selects among existing computational pathways; weights build new ones. The software/hardware metaphor is the piece's lasting contribution: frozen weights are the instruction set, context is the program running on it.

---

## Key Quotes

> Both context (via the KV cache) and weights modulate activations, but they do so through fundamentally different mechanisms with different tradeoffs.

This is the thesis that drives the whole argument. It sounds obvious but the field treats context as a cheap, fast substitute for weight updates — Jayendran's point is that substitution is architecturally limited, not just pragmatically inferior.

> A model cannot silently miss something in its own weights. It can miss a retrieval.

The cleanest articulation of the asymmetry. Weights are always active; context requires the right retrieval. This is framed as a feature of weights (reliability) but doubles as a feature of context (pluggable, inspectable, ephemeral). The tradeoff cuts both ways.

> When target behaviour requires internal representations that pretraining never developed because the data contained nothing like them, no amount of context can conjure those representations.

The ceiling statement. The long tail of human specificity — codebase conventions, regulatory nuance, team jargon, how an individual thinks — is definitionally outside the pretraining distribution. Jayendran argues this isn't exotic; it's the everyday work of adaptation.

> Weight modification adds new instructions to the architecture. It's not writing a longer program. It's redesigning the chip.

The metaphor at its punchiest. A LoRA adapter is not "more context compressed" — it's qualitatively different. Context emulates missing instructions; weight modification adds them.

> When something moves from software to hardware, from interpreted to native, the efficiency difference *is* the point.

Jayendran anticipates the objection that O(n) KV cache attention is "just an engineering problem" and preempts it: the magnitude of the efficiency gap (kilobytes vs. millions of tokens, O(1) vs. O(n), compounding vs. one-shot) means it's a qualitative difference, not a quantitative one.

> The formal separation is an open research question.

Rare honesty. The author admits no clean theorem proves weight modification's function class strictly contains context modulation's. What exists is Von Oswald's result (one gradient step = bounded move in function space), consistent empirical evidence, and architectural reality.

> The solution wasn't to make the hippocampus infinitely large.

Evolution's answer — two memory systems with different properties, connected by a transfer policy (sleep consolidation) — is both the cleanest summary and the best argument for "both."

---

## Key Themes

#concept #architecture #memory

**The Software/Hardware Distinction.** The central metaphor is unusually good: it maps cleanly onto the technical reality and explains why O(1) vs. O(n) isn't just an optimization problem. It builds on [[Smart Models Dumb Pipes]]'s end-to-end framing — the model *is* the hardware, context *is* the software — and extends it inward to the model's own architecture.

**Two Memory Systems.** Jayendran's KV cache (working memory) vs. weights (long-term memory) taxonomy parallels [[Memory Mechanism]]'s five-type framework, but at the architecture level rather than the application level. The hippocampus/neocortex parallel connects to [[Engineering the Substrate]]'s first-person account of subcortical memory vs. RAG, and to [[Three Tier Memory]]'s practical hot/warm/cold hierarchy.

**Von Oswald's Result.** The one-step gradient descent equivalence for linear self-attention is the mathematical spine of the argument. It proves that ICL is learning — but also that it's bounded, one-step learning. [[Self-Distillation]] explores what happens when you chain these steps; Jayendran's point is that one step isn't enough for distribution shift.

**The Ceiling Is Mundane.** The things that fall outside pretraining distribution aren't exotic — they're the specific, the local, the institutional. This connects to [[How Hightouch Built Their Long-Running Agent Harness]]'s finding that context management (not model ability) is the real engineering challenge, and to [[Context Rot]]'s documentation of what happens when retrieved context degrades.

---

## Critical Analysis

This is one of the best-written pieces on context vs. learning I've encountered. Jayendran does three things that are rare in this discourse: he engages the strongest version of the opposing argument (not a strawman), he acknowledges where his own position lacks formal proof, and he uses a metaphor that clarifies rather than obscures.

The software/hardware metaphor is genuinely useful, not decorative. It explains the efficiency gap as architectural rather than incidental, it makes the Von Oswald result intuitive (one instruction executed vs. a new instruction added), and it forces the question: are you building software or redesigning the chip? The answer changes the architecture.

But the piece has blind spots. It's silent on test-time training and other techniques that blur the weight/context boundary. LoRA adapters loaded at inference time are "weight" modifications that behave like "context" in their pluggability — the binary Jayendran draws is cleaner than practice allows. The [[Subquadratic 12M Context Window]] work, if it pans out, could change the efficiency math in ways Jayendran dismisses as "just engineering."

The hippocampus/neocortex parallel is apt but underdeveloped. Sleep consolidation as a transfer policy from working memory to long-term memory is the most interesting idea in the piece, and it gets one sentence. What would a sleep-like consolidation process for LLMs actually look like? [[Self-Distillation]] and periodic fine-tuning on accumulated context are approximations, but neither captures the selective, generative nature of biological consolidation.

The strongest move is Jayendran's framing of "the ceiling" as mundane rather than exotic. This is a genuinely important argument: the long tail of human specificity is the everyday work, not the edge case. If context alone can't reach it, then context alone is insufficient — not for exotic reasons, but for the most common ones.

Ultimately, "both" is the right answer, and Jayendran makes the case better than anyone else I've read. The question isn't whether to use context or weight updates — it's what transfer policy connects them.

---

*Sources: [[raw/context-is-not-learning]]*
*Last updated: 2026-05-15*
