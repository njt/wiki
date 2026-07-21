# Holding the LLM Stack in Your Head

Nick Gustafson's ambitious ~84-post series walking the full modern LLM stack in dependency order: from the linear algebra under a single attention head through training, inference engines, retrieval, and agent protocols. Written as a public learning exercise in collaboration with Claude Opus 4.8, it's a first draft that prioritizes intuition-building over reference completeness — and is more useful for it.

---

## What It Covers

The series spans twelve arcs, each a dependency-ordered sequence of posts:

1. **Mathematical & Computational Prerequisites** — vectors, matrices, norms, softmax, cross-entropy, gradients, optimizers, GPU floating point, and a short prehistory of statistical NLP. The foundation arc: everything else assumes you can read these concepts fluently.
2. **Language Modeling Before Transformers** — the pre-transformer lineage: n-grams, RNNs, LSTMs, and what transformers replaced and why.
3. **Tokenization & the Input Pipeline** — how text becomes tokens, why tokenization choices cascade through everything downstream.
4. **Transformers from First Principles** — the marquee arc: self-attention, multi-head attention, positional encoding, residual connections, layer norm. "The minimum path to really seeing how a transformer layer works, not just reciting the shapes."
5. **Decoding & the Real Inference Algorithm** — prefill vs. decode, the one-new-row insight, KV cache, PagedAttention. Why inference is slow and what the systems built around the KV cache actually do.
6. **Inference Engines & Serving Systems** — vLLM, continuous batching, quantization at serving time, the economics of GPU memory bandwidth.
7. **Training & Post-Training** — pretraining, SFT, RLHF, DPO, and everything that happens after the base model exists.
8. **Evaluation & Scientific Discipline** — benchmarks, contamination, the difference between measuring a model and understanding it.
9. **Retrieval, Memory & Context Engineering** — embeddings, chunking, reranking, RAG architectures end to end. "Just enough retrieval theory to make the engineering decisions actually make sense."
10. **Tools, Protocols & Agent Loops** — function calling as structured generation, the agent loop (model → runtime → tool → resume), MCP as cross-system standard, the history of agents from ReAct to 2026.

## Key Quotes

> "It's intuition that survives contact with real systems."

The series thesis in one sentence. Gustafson is building mental models, not reference documentation — the kind of understanding that doesn't shatter the first time you stare at a real inference trace or debug a tokenization mismatch.

> "Me thinking out loud with a model to understand the stack end to end."

This disarmingly honest framing is worth flagging because it's also the series' hidden strength. Gustafson isn't pretending to authority; he's documenting a collaborative learning process with Claude Opus 4.8, and the result reads like the best kind of pair-programming session — two intelligences working through a curriculum neither could have produced alone. The grain-of-salt caveat is correct (AI-assisted learning exercises should be treated as unverified), but it undersells what's here: a coherent, sequenced, dependency-aware curriculum that few human-authored textbooks have achieved.

## Key Themes

- **#concept — Dependency-ordered curriculum design**: The series' real innovation isn't any single explanation — it's the dependency ordering. Each arc builds on concepts introduced in prior arcs, and each post within an arc chains to the next. This is the curriculum-design equivalent of [[The Lindy Effect in Software]]: structure that survives because it matches how the concepts actually depend on each other, not because it follows textbook convention.
- **#pattern — Intuition over formalism**: Gustafson explicitly targets "intuition that survives contact with real systems," a framing that echoes [[Engineering for Bounded Cognition]] — working memory holds ~4 chunks, and the series' structure is a prosthetic for building one chunk at a time.
- **#tool — AI as collaborative curriculum designer**: The series is itself a worked example of the thing it teaches. An LLM helped structure and write a curriculum about LLMs. This isn't circular — it's a demonstration that the right collaboration pattern (human sets the dependency graph and curates, model fills in explanations) produces better pedagogy than either could alone. Related: [[What You Bring to AI Determines the Result]].
- **#concept — The full-stack imperative for LLM practitioners**: The series argues by structure that you can't understand inference engines without understanding attention, can't understand attention without linear algebra, and can't build agents without understanding the full pipeline. This mirrors the thesis of [[The Hitchhiker's Guide to Agentic AI]] — you need every layer. But Gustafson's version is a curriculum, not a reference; it's designed to be read in order rather than consulted.

## Critical Analysis

**What makes this different from other LLM educational resources.** The comparison that matters is [[Learn AI Layer by Layer]], Rob Ennals' equally ambitious interactive tutorial built for his 11-year-old son. Both are dependency-ordered from-the-ground-up explanations of the same stack. But they diverge sharply in method: Ennals builds interactive browser playgrounds where you tune parameters and watch the math respond; Gustafson builds prose explanations through dialogue with an LLM. The Ennals approach is more verifiable (the widgets either match reality or they don't), but the Gustafson approach is more scalable (you can produce 84 posts faster than you can build 84 interactive widgets). The right answer is probably both: read Gustafson for the narrative arc, then test your understanding with Ennals' widgets.

**The AI-collaboration disclosure is both honest and a dodge.** Gustafson is transparent that Claude Opus 4.8 was his collaborator, which is more than most AI-assisted educational content discloses. But "take it with a grain of salt" puts the verification burden on the reader without providing the tools to do it. The series has no citations, no links to primary sources, and no mechanism for readers to flag errors. The missing piece is a community verification layer — a way for the corrections Gustafson acknowledges he needs to come from readers rather than requiring him to "grind through polish" alone.

**The curated learning paths are the most valuable artifact.** The four paths (Understanding Attention, Why Inference Is Slow, Building with RAG, Building Agents) each pull 3–4 posts from different arcs into a focused reading sequence. This is better curatorial work than the arcs themselves — it recognizes that most readers don't need the full stack, they need a specific operational understanding. The paths model a better way to structure technical education: write the comprehensive version for completeness, then publish the curated paths for actual use.

**The relationship between this and AI-assisted education more broadly.** Gustafson's series is a case study in the pattern [[Context Is Not Learning]] identifies: the LLM can produce clear explanations, but the understanding has to be built in the human's head through structured exposure to concepts in the right order. The series' dependency ordering is the human contribution; the prose is the AI's. The curriculum is the durable artifact; the explanations are disposable and rewritable as models improve. This is the [[Specifications as the Product]] pattern applied to education: own the structure and the learning objectives; let the model fill in the exposition.

## Gaps and Limitations

- **No interactive components.** Unlike [[Learn AI Layer by Layer]], there's nothing to click, tune, or run. Prose explanations of attention without a playground to watch attention weights shift is exposition without verification.
- **Unverified AI-generated content.** The collaboration disclosure is honest, but the series operates at a trust-me level that would not survive in any field with formal peer review. This is fine for a learning exercise; it would be irresponsible for a reference.
- **Draft status acknowledged but unaddressed.** "A full first draft" with plans to polish is a commitment that most side projects never fulfill. The series' value is in what exists now, not what might exist later.
- **No coverage of mechanistic interpretability.** Given the series' "from first principles" ethos, the absence of [[Global Workspace in Language Models]], [[Jacobian Lens]], and the broader interpretability literature is a structural gap — understanding what happens inside the model is as foundational as understanding the architecture.

---

*Sources: [[raw/gustafson-llm-stack-series]]*
*Last updated: 2026-07-21*
