# The Hitchhiker's Guide to Agentic AI

Haggai Roitman's 603-page practitioner's reference — essentially a book — that surveys the entire agentic AI stack from transformer architecture through production deployment. The organizing thesis: you can't build great agentic systems by knowing only your layer; you need to understand every layer of the pipeline.

---

## Key Themes

### The Full-Stack Thesis

Roitman's argument is structural, not aspirational. Agentic systems fail at the boundaries between layers — a brilliant harness can't save a model that was fine-tuned on the wrong data, and a brilliant model can't survive a harness that loses context. The book's length is the argument: each layer deserves rigorous treatment because each is a failure surface.

This distinguishes it from most agent references, which focus on a single layer (prompting, or orchestration, or evals) and gesture at the rest. Roitman treats the stack as indivisible.

### Two Halves, One Stack

**Part 1 — The LLM Substrate:** Transformer architecture, GPU systems, training and fine-tuning (SFT, LoRA, MoE), model compression, inference optimization. Then alignment and reasoning: RLHF, PPO, DPO and variants, GRPO, reward modeling, RL for large reasoning models including chain-of-thought and test-time scaling.

**Part 2 — Agentic Systems:** Agentic training and trajectory-based RL, RAG and Agentic RAG, memory systems, agent harness design, a taxonomy of agent design patterns. Inter-agent coordination: MCP, agent skills and tool use, A2A communication protocol, multi-agent architectures (centralized, decentralized, hierarchical). Closes with development frameworks, agentic UI design, evaluation methodology, and production deployment.

### Theory + Implementation + Code

Each chapter follows a three-part structure: rigorous theoretical foundations, implementation guidance, and code examples with references. This is the pattern that separates a usable reference from a literature survey — Roitman doesn't just tell you what exists, he tells you how to build it.

---

## Critical Analysis

**What this is:** The closest thing the agentic AI field has to a textbook. 603 pages covering the full stack with implementation guidance is unprecedented. Most practitioners operate with fragmentary knowledge — they know their framework, their model, or their eval methodology, but not how they connect. Roitman's reference fills that gap.

**What it isn't:** A quick read. 603 pages is a commitment, and the "every layer" thesis means you can't skip around arbitrarily — the layers build on each other. This is a reference to grow into, not a blog post to scan.

**The structural bet:** Roitman is betting that agentic AI is mature enough to deserve a textbook but young enough that one person can still credibly cover the whole stack. That window is closing fast — specialization is accelerating, and in two years a single-author comprehensive reference may be impossible. This book catches the field at exactly the right moment.

**The open-access play:** CC BY-SA 4.0 on arXiv means this can be remixed, translated, and adapted. In a field where the best references are often locked behind paywalls or scattered across blog posts, an open-access textbook is a public good. The license choice is as important as the content choice.

**What's missing:** A 603-page survey can't also be deep on every topic. The tradeoff is breadth over depth — each chapter gets enough treatment to be useful but not enough to be definitive. Practitioners will still need specialized references for production deployment of any single layer.

**The implicit thesis about you:** Roitman believes practitioners should understand the whole stack. This is a normative claim — it says something about what kind of engineer you should be. It pushes back against the specialization trend and the "just use the API" mindset. Whether that's practical for most teams is debatable, but it's a worthy aspiration.

---

## Cross-References

This book intersects with nearly every page in the wiki. Key touchpoints:

- [[Elements of Agentic Systems Design]] — Chen's ten-element taxonomy is a design-space map; Roitman's book is the implementation manual for that map
- [[The Agentic Product Standard v2.0]] — the production standard operationalizes what Roitman surveys; they're complementary (survey vs. spec)
- [[AI Engineering for Developers]] — Cavallin's field guide covers similar ground at blog-post depth; Roitman provides the textbook treatment
- [[Components of a Coding Agent]] — Raschka's harness taxonomy is one chapter of what Roitman covers in full
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE is a production case study of the RAG and harness chapters
- [[Agent Memory and Context]] — the hub page for Roitman's memory systems chapter
- [[Agent Orchestration]] — the hub page for Roitman's multi-agent coordination chapter
- [[Building Agents for Production Systems with MCP]] — Anthropic's MCP guide is the applied companion to Roitman's MCP chapter
- [[Guardrails and Feedback Loops]] — the hub page for Roitman's evaluation methodology chapter

---
*Sources: [[raw/hitchhikers-guide-agentic-ai]]*
*Last updated: 2026-07-08*
