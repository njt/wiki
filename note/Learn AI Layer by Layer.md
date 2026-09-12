# Learn AI Layer by Layer

The best free AI tutorial on the internet right now. Rob Ennals built it for his 11-year-old son after finding every existing AI explainer either too vague, too presumptuous about expertise, or too boring. The result: nine published chapters (through Transformers) that build modern AI from first principles using interactive browser playgrounds — not gifs, not videos, but actual widgets where you tune parameters and watch the math respond. Middle-school math floor, no ceiling on depth.

---

## Key Quotes

> "Playing with an idea makes it easier to grasp than just reading about it."

This is the tutorial's thesis and its differentiator. Every chapter has sliders, clickable diagrams, real-time computed output. The attention chapter lets you click individual boxes in a transformer block to read what each component does. The embeddings chapter gives you a live word-similarity explorer. This isn't garnish — it's the pedagogy. [[There Is No Spoon]] makes the same bet (intuition over notation) but uses physical analogies; Ennals uses direct manipulation.

> "In a normal neural network the wiring is fixed, learned once during training. Attention dynamically determines where information should flow."

The transformer chapter's cleanest line. It names what makes the architecture different without needing the word "dynamic" — the contrast between fixed weights and routed information is the whole story. [[Talking to Transformers]] frames attention as a zero-sum budget; Ennals frames it as self-modifying wiring. Same insight, different audience.

> "Softmax always allocates 100% somewhere, even when no real match is available."

The attention chapter's sink-token discussion is where Ennals earns his keep. Most tutorials hand-wave softmax as "converts to probabilities." Ennals shows why that's a problem: off-task tokens still get 100% allocated somewhere. The sink token fix isn't presented as an implementation detail — it's presented as the shape of the problem. This is what separates "explaining" from "building intuition."

> "The dimensions don't need labels. What matters is that similar things are nearby and that directions encode meaningful relationships."

The embeddings chapter's core insight, stated plainly enough for a middle-schooler but deep enough that most practitioners forget it. You don't label dimension 47 as "nouniness" — you just let training place "cat" near "dog" and discover that "elephant − mouse" points toward "lion." [[A Non-Anthropomorphized View of LLMs]] argues the same from the other direction: these are functions through ℝⁿ, not minds with labeled concepts.

> "We usually don't actually know what the intermediate representations of tokens mean."

Ennals is honest about the limits. After nine chapters building up the transformer as this elegant, comprehensible architecture, he admits: we can't read the thing we built. The representations aren't optimized for human understanding — they're optimized for next-word prediction. This is where the tutorial hands off to the research frontier (mechanistic interpretability, [[Emotion concepts and their function in a large language model]]).

## Architecture of the Tutorial

The nine published chapters form a strict dependency chain — each concept builds on the last:

1. **Computation** → everything is numbers; models are machines with knobs
2. **Optimization** → gradient descent as incremental improvement (evolution, A/B testing, science)
3. **Neural networks** → neurons as smooth logic gates; XOR as the crisis that proved depth matters
4. **Vectors** → the dot product; a neuron IS a dot product with an activation function
5. **Embeddings** → unlabeled dimensions; vector arithmetic; subword tokenization
6. **Next-word prediction** → prediction requires understanding; n-gram explosion; Friston's free energy principle
7. **Attention** → Q/K/V; softmax; sink tokens; multi-headed attention as parallel questioning
8. **Positional encoding** → ALiBi vs. RoPE; rotation trick; why every frontier model uses RoPE
9. **Transformers** → stacked layers, dynamic routing, 96 layers × 96 heads × 12,288 dims

Then appendixes on PyTorch and a glossary. Chapters 10–27 (matrix math, MoE, RL, reasoning, alignment, agents, world models) are "coming soon."

The PyTorch appendix is genuinely useful: tensors, autograd, `nn.Linear`, the training loop, a complete XOR example. All Colab-based — no local install needed, free GPU.

## Key Themes

#tool #concept #tutorial #person #visualization

- **Interactive-first pedagogy**: The widgets aren't decoration. The attention playground where you step through layers one at a time and watch token representations evolve is doing work that prose can't.
- **Middle-school math floor**: Ennals takes this constraint seriously without letting it become a ceiling. The sigmoid explanation works for a 12-year-old; the Church-Turing thesis reference works for an adult.
- **Honesty about what we don't know**: The interpretability admission in the transformer chapter is rare in educational material. Most tutorials present the architecture as solved; Ennals presents it as working-but-opaque.
- **The sink token as microcosm**: The attention chapter's treatment of "what happens when nothing matches" is the tutorial at its best — identifying a real sharp edge and solving it in view of the reader.

## Critical Analysis

This tutorial does something genuinely unusual: it's accessible to an 11-year-old and I'd recommend it to a senior engineer who wants to build intuition about why transformers work the way they do. That's a narrow target to hit and Ennals hits it.

The interactive playgrounds are the moat. You can read about softmax in a hundred blog posts. You can watch the percentages redistribute as you crank the query magnitude in only this one. The difference is the difference between reading about riding a bike and falling off one.

**What it sacrifices**: Completeness. Stopping at transformers (chapter 9 of 27 planned) means the tutorial doesn't touch RLHF, reasoning models, agent architectures, or any of the post-training techniques that actually make frontier models useful. The "coming soon" list is ambitious — world models, audio, agents, hallucination — but right now the tutorial is a foundations course, not a survey.

**What it nails**: The embeddings → attention → positional encoding → transformer sequence. This is the conceptual spine of modern AI, and Ennals builds it with the patience of someone who actually tested each explanation on his kid. The Q/K/V decomposition with sink tokens is the best short treatment of attention I've read.

**Comparison to [[There Is No Spoon]]**: No Spoon targets engineers who can whiteboard software systems — it's deeper on specific insights (paper-folding model of depth, combination rules as architecture choice) but narrower in scope. Layer by Layer targets anyone with middle-school math and covers the full stack from numbers to transformers. No Spoon's analogies are more memorable; Layer by Layer's interactives build more operational understanding. Read both.

**The "coming soon" problem**: Honest but concerning. Writing nine chapters with interactive playgrounds and Colab notebooks is real labor. 18 more chapters is a lot of labor. The Substack subscription model suggests Ennals has a plan for sustainability, but educational projects this ambitious have a high mortality rate. What exists now is worth reading regardless.

**What I'd add**: A chapter on "what happens after the transformer" — even a short bridge explaining that GPT isn't just a transformer doing next-word prediction, it's a transformer trained on next-word prediction and then fine-tuned with RLHF to be useful. The tutorial's current endpoint risks leaving readers with the "just predicts the next word" misconception that chapter 6 explicitly argues against.

---

*Sources: [[summary/learn-ai-layer-by-layer]]*
*Last updated: 2026-05-31*
