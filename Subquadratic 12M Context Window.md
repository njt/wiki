# Subquadratic 12M Context Window

A 13-person Miami startup claims to have broken the Transformer's quadratic bottleneck with a sparse attention architecture that scales linearly to 12 million tokens at 5% the cost of Claude Opus. If real, it's a paradigm shift. The evidence is thin: no paper, no open weights, self-reported benchmarks.

---

## Key Quotes

> "We are very focused on the problem of how we transition from a dense attention, quadratic scaling architecture to a sparse attention linear architecture."
> — CEO Justin Dangel

The diagnosis is correct. The question is whether they've actually solved it.

> "If you double the input size with quadratic scaling laws, you need four times to compute; with linear scaling laws, you need just twice."
> — CTO Alexander Whedon

This is the mathematical promise. Sparse attention is not new — the research direction is legitimate. What's unproven is whether SubQ achieves this without sacrificing reasoning quality on tasks more demanding than token recall.

> "The fundamental scaling laws imposed by the transformer architecture and dense attention have been broken through."
> — Dangel

Extraordinary claims, extraordinary evidence required. We have a tech blog post and self-reported numbers.

> "I used to manually curate prompts and retrieval systems and evals and conditional logic to chain together the workflows... a waste of human intelligence."
> — Whedon, on why RAG and agentic retrieval are stopgaps

This is the most honest framing in the piece. If Whedon is right, SubQ makes most of the wiki's context management pages historical footnotes. That's a big if.

## The Numbers

| Benchmark | SubQ | Best Frontier | Notes |
|---|---|---|---|
| RULER @ 128K | 95% ($8) | Opus: 94.8% (~$2,600) | The headline number. 300x cheaper. |
| SWE-Bench Verified | 81.8% | Opus 4.7: 87.6% | Close but not leading. The coding benchmark that matters. |
| MRCR v2 (1M tokens) | 65.9% | Opus 4.6: 78.3% | Multi-hop reasoning at length. SubQ lags significantly. |
| 12M token recall | 92% | N/A | No one else can play. But recall is the easiest test. |

The RULER cost comparison is the number they lead with. SWE-Bench and MRCR tell a different story: competitive but not dominant, and worse on reasoning that requires connecting information across the full span.

## Products

- **SubQ API** — full 12M-token context window
- **SubQ Code** — CLI coding agent that loads an entire codebase in one pass, removing the need for retrieval, chunking, or multi-agent coordination
- **SubQ Search** — deep research tool, initially free (land-and-expand play)

SubQ Code is the most interesting product thesis. If it works, it eliminates [[Context Rot]], [[Agent Memory and Context]] chunking strategies, [[Slate]]-style context routing, and most of the retrieval pipeline the entire agent ecosystem is built on. If it doesn't, it's a demo that falls apart the moment you ask it to reason across the full codebase rather than just recall lines from it.

## Key Themes

#concept (sparse attention, sub-quadratic scaling) #tool (SubQ, SubQ Code) #architecture (SSA as Transformer alternative) #person (Justin Dangel, Alexander Whedon) #comparison (Transformer vs. SSA, dense vs. sparse attention)

## Critical Analysis

**The good case.** Sparse attention is a real research direction. The team's PhDs are from Meta, Google, Oxford, Cambridge, Adobe — not randos. The backers include early Anthropic and OpenAI investors who know what due diligence means. If linear attention works at scale, the economics of long-context AI invert: RAG becomes a legacy pattern, not a fundamental requirement.

**The bad case.** This is a 13-person company with no published paper, no open-source code, and all benchmarks self-reported. The 300x cost comparison on RULER is a cherry-picked number — on SWE-Bench the margin is 6 points behind Opus 4.7, and on MRCR (which actually tests reasoning across long context) SubQ scores 12 points lower than Opus 4.6. The @ 128K on RULER is also doing work: at 128K tokens, Frontier models aren't even in their weak-scaling regime yet. Show me the numbers at 1M+ tokens where quadratic attention actually hurts.

**The tell.** The MRCR v2 gap is the most honest number they've published. Multi-hop reasoning across long context is the hard problem, and SubQ is worse at it than models with 12x smaller context windows. If your architecture trades reasoning depth for context breadth, you've built a very fast retrieval engine, not a reasoning model.

**The coding agent thesis.** SubQ Code claims to load an entire codebase into context. This is the dream: no chunking, no retrieval, no lost context. But the SWE-Bench score (81.8%) suggests it's not actually better at the thing that matters — fixing bugs across a codebase — than models that use retrieval. A model that can *see* the whole codebase but can't *reason* across it is a party trick, not a tool.

**The infrastructure angle.** Even if SSA works, 12M tokens at 150 t/s decoding means SubQ is still slow for interactive use. Prefill speedups don't help when you're waiting for the model to type. This is the same problem [[How AI Labs Are Solving the Power Crisis]] attacks from the hardware side — SubQ is an algorithmic answer to the same cost curve.

**Bottom line.** Assume it's real but not as good as the headline numbers suggest. Sparse attention is probably part of the future architecture mix. SubQ probably isn't the final form. The coding agent claim (load entire codebase, no retrieval needed) is the one to watch — if it holds, it reshapes the agent architecture landscape the wiki catalogs. Wait for independent benchmarks before updating your mental model.

---

*Sources: [[raw/subquadratic-12-million-context-window]], SiliconANGLE (2026-05-05)*
*Last updated: 2026-05-15*
