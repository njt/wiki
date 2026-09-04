# Scaling RAG — Chunking, Reranking, and Cost Optimization

A practitioner's field guide to running retrieval-augmented generation in production, built from the failure cases: systems that work on toy test sets and "fall apart" on real data. The argument is that RAG-as-"throw-documents-in-a-vector-DB" is a prototype, not a system — and that the difference between working and frustrating RAG is priorities, not talent: be precise about what you measure, ruthless about what you optimize, and honest about what doesn't work for your problem.

---

## Key Quotes

> "The problem isn't RAG itself. It's that most teams treat RAG as 'throw documents in a vector DB and ask questions.' That's not RAG at scale. That's a prototype."

The thesis. It frames everything downstream: production RAG is a hundred small decisions (chunking, embedding model, reranking, retrieval), not a single architectural choice.

> "Your chunking strategy determines everything downstream. Get it wrong and no amount of reranking or retrieval magic fixes it."

Chunking is the highest-leverage decision — the same claim [[Production RAG in .NET]] makes from the .NET/Semantic Kernel ecosystem. Fixed-size chunks (512 tokens, 50 overlap) split sentences mid-thought and ignore document structure; the fix is structure-aware chunking: split on markdown headers first, chunk semantically within sections, attach metadata.

> "This is where 73% of RAG failures happen. You retrieve garbage, the LLM can't fix it."

The number that justifies hybrid retrieval. Vector-only got 64% recall on a real project, BM25-only 71%, and the combination 82% — the two techniques catch edge cases the other misses.

> "The large model won't save you if your chunks are garbage."

The cost/quality corollary: a local embedding model that's 1–2% worse at retrieval is fine once a reranker is in the loop, so spending on a large embedding model before fixing chunking is money spent in the wrong place.

> "Better quality, 5.7x cheaper. Tradeoff: operational complexity."

The cost table's punchline. On a 500k-document system the "expensive" stack (text-embedding-3-large + Pinecone Pro + Cohere rerank) ran $684/month at 78% recall; the "optimized" stack (local embeddings + self-hosted Weaviate + local reranker) ran $120/month at 81% recall. Cheaper *and* better — the catch is that you now operate the machinery yourself.

---

## Key Themes

#pattern #RAG #retrieval #chunking #reranking #cost-optimization #vector-search #production-engineering

- **Chunking is a quality lever, not boilerplate.** The post's three-tier progression (fixed-size → semantic → structure-aware production chunking) mirrors the pattern across the wiki: [[Chunker (Document Chunking Tool)]] uses LLM-mediated semantic boundaries instead of fixed-size splits, and [[Production RAG in .NET]] argues paragraph-aware splitting beats fixed character counts.
- **Retrieval is the failure point, not generation.** 73% of failures originate before the LLM ever sees context, which is why hybrid vector+BM25 with reciprocal rank fusion is worth the extra moving part. Reranking then fixes the ordering — a 10–30% quality gain for 50–200ms of cross-encoder latency.
- **Cost is a stack decision, not a model decision.** The 5.7× cost reduction comes from swapping managed services for local models and self-hosted retrieval, not from a cheaper LLM. This rhymes with [[Just Brute Force Your Embeddings]] (measure before you architect) and [[Binary Vector Embeddings]] (compression as a cost lever), but pushes further: the expensive stack here is *also* lower quality because it papers over bad chunking with a big model.
- **Debugging is prioritized and checklist-driven.** Three playbooks — irrelevant retrieval, >500ms latency, growing cost — each diagnose the bottleneck in order before touching anything. The cost one lands a non-obvious point: LLM generation dominates (~60%), so shrinking context from 2000 to 1000 tokens saves more than optimizing reranking.

---

## Critical Analysis

The strongest thing about this post is its honesty about *numbers* — 64/71/82% recall, $684 vs $120, 10–30% reranking uplift, 60% cost share for the LLM. That's a refreshing contrast with RAG material that waves at "better retrieval" without a single measurement. The weakest thing is the code: it's pedagogical pseudocode dressed as production code. `semantic_chunking` splits on `. ` which mangles "Mr. Smith" and "e.g."; `fixed_size_chunking` operates on *characters* while the prose talks about *tokens*; the cost calculator hardcodes prices that will drift. None of this invalidates the advice, but it undercuts the "what I learned from running RAG systems that actually work" framing — you'd hope someone who ran a 500k-document system would show the splitter they actually shipped rather than a 30-line toy.

The cost table deserves scrutiny too. The "expensive" and "optimized" stacks differ in three variables at once (embedding model, vector store, reranker), so the 5.7× figure bundles model cost, managed-service markup, and reranking API fees into one number. It's directionally right — local embeddings plus a self-hosted store plus a local reranker will be much cheaper — but it's not a controlled comparison, and "operational complexity" is doing a lot of quiet work as the residual cost. Someone has to keep Weaviate healthy and the GPU warm, and that person isn't free.

The quiet claim worth pulling out: **better chunking improved recall more than switching to a large embedding model did.** The post reports going from 78% to 81% recall *with* the cheap stack because chunking + hybrid retrieval + local reranking beat a large embedding model over garbage chunks. That's the load-bearing insight, and it aligns with [[Production RAG in .NET]]'s insistence that "most RAG problems are not solved by adding more infrastructure." It also sets up the natural synthesis with [[GraphRAG]]: the post treats "throw documents in a vector DB" as the baseline to escape, and GraphRAG is one structured alternative — but this post's escape route (better chunking + hybrid retrieval + reranking) is dramatically cheaper than GraphRAG's LLM-heavy indexing, and should probably be tried first.

What the post gets right that's easy to miss: reranking is presented as a *gateable* stage, not a default. The `ProductionRAG` config makes `enable_reranking` a toggle, and the diagnostic playbook says "if not in top-20 at all, problem is in retrieval, not ranking." Reranking can't rescue retrieval that never surfaced the right chunk — a boundary most teams learn the expensive way.

---

*Sources: [[raw/scaling-rag-chunking-reranking-and-cost-optimization]], [[summary/scaling-rag-chunking-reranking-and-cost-optimization]]*
*Last updated: 2026-09-04*
