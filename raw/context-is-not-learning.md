---
url: https://www.aravindjayendran.com/writing/context-is-not-learning
date_fetched: 2026-07-05
backfilled: true
---

The default answer to "how do we make LLMs learn?" (continual learning, not training from scratch) is: make context windows longer. Opus 4.7 has 1M tokens. KV cache compression keeps improving. Linear attention variants are making long context computationally cheap. The implicit assumption behind all of this: if context gets long enough and cheap enough, you don't need the model to update its own weights.

This assumption is incomplete in a way that matters. To see why, we need to look at what context and weights actually do inside a transformer. They're more similar than most people realise, and the difference between them is more fundamental than "one is temporary and the other is permanent."

## What context and weights actually do

Every layer in a transformer produces activations: internal representations that flow forward to the next layer and ultimately determine the output. Both context (via the KV cache) and weights shape these activations. They're doing the same job through different mechanisms.

When you fine-tune a model, you change its weights. This changes how every input gets transformed at every layer. The activations shift toward a new distribution. Show the model enough examples of legal contracts, and its internal representations reorganise to process legal language more effectively. This shift is permanent.

When you do in-context learning, the KV cache fills with key-value pairs from your context. These cached representations influence how the model processes subsequent tokens through attention. The activations shift, sometimes dramatically. Few-shot prompting works precisely because those cached examples steer the model's internal computation toward the demonstrated pattern. But clear the context, and the activations revert to their default state.

Same job. Different mechanism. One permanent, one temporary.

This equivalence isn't just conceptual. Von Oswald et al. (2023) proved that for linear self-attention, the activation shift from in-context learning is mathematically equivalent to one step of gradient descent, the same operation used in fine-tuning. The KV cache is, in a real sense, a transient weight update. Mahankali et al. proved this is optimal for one-layer linear transformers.

So if they're doing the same thing, why does it matter which one we use?

## Software vs hardware

Think of the frozen weights as hardware: they define what computations the model can perform, what patterns it can recognise, what representations it can build. Context is software running on that hardware.

A well-designed general-purpose processor can run a huge variety of programs. x86 handles word processing, video rendering, ML inference. The instruction set is rich enough to express almost anything. Similarly, a well-pretrained LLM handles an impressive range of tasks through in-context learning: translation, reasoning, code generation, style transfer.

But software has limits that hardware doesn't. If the chip lacks a floating-point unit, your software float emulation works, but it's slow and limited in precision. If the processor lacks SIMD instructions, your matrix multiply runs, but it's orders of magnitude slower than dedicated silicon. The program can be arbitrarily long. The instruction set is fixed.

Weight modification adds new instructions to the architecture. It's not writing a longer program. It's redesigning the chip.

## The strongest case for longer context

Before arguing that weights matter, we should honestly engage with why longer context is so compelling.

Modern LLMs aren't random chips running arbitrary programs. They're specifically designed for in-context learning. During pretraining on trillions of tokens, the model's weights and KV cache co-evolve. The model learns to be a powerful meta-learner: its weights are optimised to make context-based activation shifts as expressive as possible. The "instruction set" was designed with this exact use case in mind.

This means a well-pretrained model's "hardware" supports a remarkably wide range of in-context "programs." Within the space of tasks the pretraining data covered, ICL can be highly expressive. For most of these tasks, you might genuinely not need weight changes.

You could also argue that persistence (context is temporary, weights are permanent) is just an engineering problem. Prefix caching, KV serialisation, or simply recomputing from stored text all give you persistence without touching weights. And there's an interpretability advantage: context is human-readable. You can inspect what the model was told. Weight changes are opaque.

These are real arguments. The "just make context longer" position is stronger than most weight-space advocates admit. Letta's "Continual Learning in Token Space" makes this case explicitly, and it's worth reading even if you disagree.

## The ceiling

But the frozen weights are, in effect, a fixed meta-learning algorithm. Pretraining optimised that algorithm for the pretraining distribution. Within that distribution, it's powerful. At the boundary, it hits a ceiling.

When the target behaviour requires internal representations that pretraining didn't develop, because the data didn't contain them, no amount of context can conjure those representations. The program can be infinitely long. If the instruction set doesn't have the right primitives, it can't compute the target function.

Think about what falls outside the pretraining distribution. Not exotic things. Mundane, specific things: the architectural conventions of your particular codebase, the regulatory nuances of your specific industry, how you personally think about problems, the internal jargon of your ten-person team. General pretraining covers general knowledge well. The long tail of human specificity is, by definition, not well-covered.

This is where weight modification matters most. It doesn't just run a different program on the same chip. It adds new circuits. Internal representations that didn't exist before. Computational pathways the pretrained meta-learner never developed. The model doesn't just behave differently; it becomes capable of computations it couldn't perform before.

Fine-tuned models consistently outperform prompted models on distribution-shifted tasks, even with very long context. This is the empirical signature of the ceiling.

## Even within the ceiling, weights win

Grant, for the sake of argument, that context can express everything weights can. The cost is still fundamentally different.

**Inference cost.** Knowledge in weights is O(1) per forward pass. It's baked into the computation. Context-based knowledge requires attending over the full KV cache on every token generation: O(n), where n is the context length. For a million-token context, every single output token pays the cost of attending over a million cached entries.

**Compression.** A LoRA adapter encoding a complex domain adaptation is kilobytes. The equivalent context, enough examples to get the same behaviour through ICL, might be millions of tokens. Same function, orders of magnitude less storage.

**Composability.** Weight updates compound. Step 1 changes the model's computation. Step 2 operates on the *new* computation. Each update builds on the last, navigating to regions of function space that a single context window, no matter how long, cannot reach. This is the practical consequence of the Von Oswald result: in-context learning approximates one step of gradient descent. Real learning takes many steps, each compounding.

These aren't engineering quibbles. When something moves from software to hardware, from interpreted to native, the efficiency difference *is* the point. Nobody dismisses GPUs as "just an optimisation over CPU matrix multiply." The speed difference enables qualitatively different applications.

## An honest acknowledgment

We don't have a clean theorem proving that the function class reachable by weight modification strictly contains the class reachable by context modulation for well-trained models. The formal separation is an open research question.

What we have: Von Oswald's result (ICL approximates one gradient descent step, and one step can only move a bounded distance in function space), consistent empirical evidence (fine-tuned models outperform prompted models on distribution-shifted tasks across model scales), and the architectural reality that context changes the *input* to frozen computation while weight modification changes the *computation itself*. Real transformers are messier than the linear attention proofs, but the gap is observable in practice.

Proving or disproving this separation formally would be a meaningful contribution to the theory of in-context learning. The practical gap is real today, even if the theory hasn't caught up.

## The answer is both

This isn't context versus weights. It's context *and* weights.

Evolution faced the same engineering constraint: fast ephemeral memory (hippocampus) is cheap but bounded, slow persistent memory (neocortex) is expensive but vast. The solution wasn't to make the hippocampus infinitely large. It was to develop a transfer policy (sleep consolidation) that moves the right things from fast storage to slow storage. The brain has both memory systems. They're complementary, not competing.

Context windows will keep getting longer. Good. They solve the working memory problem: holding the information you need for the immediate task. Weight-space learning solves a different problem: accumulating knowledge that persists, generalises, and becomes native to the model's computation. Both are necessary. Neither is sufficient.

If you're curious about how weight-space learning could actually work, I wrote about that in Language Models Are Few-Shot Learners — They Just Can't Remember: three existing research threads that, combined, point toward a concrete mechanism. The few-shot learners are coming. They just need both memory systems.
