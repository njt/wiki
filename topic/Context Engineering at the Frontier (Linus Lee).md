# Context Engineering at the Frontier (Linus Lee)

Linus Lee (Head of AI at Thrive Capital, formerly Notion) argues that the industry's obsession with bigger context windows is a failure of engineering imagination. The real design work isn't cramming more tokens into a prompt — it's building composable retrieval pipelines, pre-structuring data at write-time, and recovering the "hand feel" for where models fail at scale. A 17-minute AI Council preview that packs more framework density than most hour-long conference talks.

---

## The Bigger Context Window Is a Cop-Out

Lee opens with a provocation: throwing everything into context is the **baseline, not the solution**. It's expensive, slow, and brittle. But his real argument is more interesting than cost — it's about **engineering velocity**.

> "If you have a single agent doing a single monolithic inference… and then you want to change some part of it… you have to run all of the evals again of anything that touches this particular generation step because it's one monolithic system."

This is the argument I haven't heard made this cleanly before. Big context windows don't just cost more tokens — they make your system **unevaluable**. Change the system prompt? Re-run everything. Change a tool description? Re-run everything. The monolith is a testing nightmare disguised as architectural simplicity.

The alternative: **composable retrieval sub-agents** with defined interfaces. Each sub-agent can be optimized, evaluated, and iterated independently. This is the same argument for microservices over monoliths, applied to agent architecture. And it maps directly onto the [[StrongDM Factory Techniques]] pipeline pattern — composable stages with inspectable outputs.

## Retrieval Is a Gradient, Not a Binary

Lee's most useful framework is the **gradient of retrieval techniques**:

| Level | What | Cost | When |
|-------|------|------|------|
| Raw context dump | Everything in prompt | $$$$ | Baseline only |
| Lightweight re-rankers | Fast embedding search + scoring | $$ | High recall, moderate precision |
| Structured query writers | Text-to-SQL, query DSLs | $$ | Structured data sources |
| Retrieval sub-agents | Full agent with tools, dedicated to one source | $$$ | Complex, multi-step retrieval |
| Top-level synthesis agent | Only sees curated results | $$$$ | Final reasoning and synthesis |

> "We can spend the expensive tokens in the final model's context… or we can push some of that retrieval work down into more and more specific tools that are less and less costly, but require maybe a little more design thinking."

The key phrase is **"require maybe a little more design thinking."** That's the trade Lee is selling: design effort now for lower cost, better evals, and faster iteration later. This is the [[Loop Engineering]] mindset — designing the system that prompts agents, not just prompting agents directly.

## Context Engineering IS Search Engineering

Lee reframes the entire problem through the lens of 50+ years of information retrieval research:

> "Context engineering as a search problem… You define some space of options… and then you have some way of either narrowing down your search base or enumerating the space so that you can iteratively search through it."

He references a 1968 IBM comic on IR that captures the essence: structure information so the computer can understand it, balancing precision and recall. The classic search stack pattern — **over-retrieve for recall, then progressively narrow for precision** — maps directly onto agent context construction.

This reframing is powerful because it lets you steal from an entire discipline. The retrieval pipeline Lee describes (fast over-retrieval → re-rankers → sub-agents → parallel merging → top-level synthesis) is just a search engine with an LLM at the end. [[Building Reliable Agentic AI Systems]] does exactly this at Bayer — an 8-step RAG pipeline with cross-encoder reranking before the final model sees anything.

What's novel here isn't the techniques individually. It's the **mindset shift** from "what do I put in context?" to "how do I build the pipeline that feeds context?" The second question has answers. The first is just vibes.

## Write-Time Structure, Read-Time Power

Lee draws on his Notion experience to make a point that's obvious in databases but underappreciated in AI:

> "How you index the data plays a role in what you can do with the data."

Pre-structure data at ingestion time, and you unlock queries that are impossible against a flat text index. This is the database designer's instinct applied to agent memory — and it's the argument [[Coding Agents Continuity Not Memory]] makes from the other direction: the problem isn't storage volume, it's structure.

The speculative thread: **sparse vectors from mechanistic interpretability**. If you could use sparse autoencoders to produce embeddings where each dimension maps to a legible concept (sentiment, presence of an idea, factual claim), you could index and query in semantically meaningful ways that dense vectors can't support. Lee admits this is "academically interesting" rather than production-ready, but it's the kind of idea that reframes what's possible.

## The Observability Gap Nobody Talks About

This is Lee's most underrated point:

> "Are these runs or are these pipelines delivering results in a way that is sort of semantically successful in the way that the user would define it? Like when they want to generate a deck, is there a deck at the end of it? Or is the model kind of stumbling through and not delivering a text answer instead?"

He calls this **semantic observability** — knowing whether the output was *semantically* what the user wanted, not just whether the pipeline completed without errors. Infrastructure monitoring (did it run?) is table stakes. Semantic monitoring (did it work?) is the hard problem.

At Thrive, with tens of billions of tokens processed, Lee says you lose the "hand feel" for where models stumble. When you're running a coding agent on your laptop, you know intuitively which tools are hard and where models fail. At scale, that intuition evaporates. You need tooling to recover it.

This is the monitoring gap that [[Guardrails and Feedback Loops]] points at — evals aren't just for development, they're for production. And it's the problem that [[The Agentic Product Standard v2.0]]'s eval pyramid tries to systematize. But Lee's framing of "semantic observability" as distinct from infrastructure observability is sharper than most.

---

## Key Themes

- #concept **Composability over monoliths** — break systems into independently evaluable pieces with defined interfaces. This isn't just about cost; it's about iteration speed and testability.
- #concept **Context engineering as search** — the entire IR literature (precision/recall, iterative refinement, parallel retrieval merging) applies directly to agent context construction.
- #tool **Retrieval sub-agents** — dedicated agents that handle one retrieval task, return curated results to the top-level agent. Independently optimizable.
- #pattern **The retrieval gradient** — push work down into cheaper, more specialized components. Reserve expensive context tokens for synthesis, not search.
- #pattern **Write-time pre-structuring** — shift intelligence to indexing time. Structure data on ingest to unlock queries impossible against raw text.
- #concept **Semantic observability** — monitor whether outputs are semantically successful, not just whether pipelines completed. Recover the "hand feel" lost at scale.

---

## Critical Analysis

**What Lee gets right:** The composability argument is the strongest part of this talk. The idea that monolithic context makes systems unevaluable is a genuinely fresh framing, and it's correct. If you've ever tried to iterate on a complex system prompt and had to re-run the full eval suite each time, you know exactly what he means.

The retrieval-as-search reframing is also valuable, even if it's not novel. Stealing from IR is smart — that discipline has been optimizing the precision/recall tradeoff for decades. Most agent builders are rediscovering these ideas from scratch.

**What Lee dodges:** The elephant in the room is that **bigger context windows work**. They're getting cheaper, models are getting better at long-context reasoning, and "just put it all in context" is often the fastest path to a working prototype. Lee acknowledges this as "a fine baseline" but never engages with the possibility that it might be the *right* answer for many use cases, not just a starting point. The [[Maybe Coding Agents Don't Need a Bigger Memory]] argument suffers from the same blind spot — it's right about the problem but wrong about the urgency of solving it.

The sparse vector idea is **pure speculation**. Lee admits he hasn't worked on it in years. It's the kind of conceptually elegant idea that's been "a few years away" for a decade. Including it in a talk about practical context engineering is either intellectual honesty (showing the long-term vision) or padding (filling time with cool-but-irrelevant ideas). I lean toward the former, but it's a close call.

Lee's composability thesis takes an interesting turn with [[Attemory]]: rather than decomposing retrieval into sub-agents with separate embedding/ranking pipelines, Attemory collapses the entire stack into a single model forward pass over raw KV-cached text. The model's own attention weights ARE the retrieval signal — no embeddings, no sub-agents, no parallel retrieval merging. It's the opposite of Lee's composability prescription, yet it achieves SOTA-class results on LongMemEval-M at million-token scale. The tension is productive: composability helps when individual components are unreliable, but if a single attention pass can outperform the decomposed pipeline, Lee's engineering-velocity argument may need to account for the maintenance cost of the pipeline itself.

**What's missing:** The talk is completely silent on **evaluation**. If your system is now a pipeline of 5+ components (re-rankers, sub-agents, query writers, top-level agent), how do you evaluate each piece in isolation? What does a good eval for a retrieval sub-agent even look like? Lee says composability enables independent evaluation but never sketches what that evaluation actually entails.

Also absent: **data freshness, permissions, and security**. Retrieval pipelines touch multiple data sources. What happens when one source is stale? When permissions differ between sources? When a sub-agent retrieves PII that shouldn't be in context? These are the problems that [[How Hightouch Built Their Long-Running Agent Harness]] and [[Building Reliable Agentic AI Systems]] actually wrestle with.

**The bottom line:** This is a talk that's better at naming problems than solving them, but the problems it names are real and under-discussed. The composability argument alone is worth the 17 minutes. The semantic observability gap is the thing everyone building at scale knows about but nobody has a good answer for. Lee doesn't have one either, but at least he's asking the right question.

**Verdict:** A dense, useful framework talk that's one notch more practical than the "context windows are expensive" discourse and one notch more conceptual than a production postmortem. Worth reading if you're building agents, essential if you're building agent infrastructure.

---

## Related

- [[Agent Memory and Context]] — Hub page for context engineering patterns
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE: the most detailed public field report on context engineering + harness engineering in production
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — Santi's argument that context ≠ continuity; repo-local continuity records beat bigger windows
- [[Coding Agents Continuity Not Memory]] — The continuity primitive: resume → work → finalize lifecycle
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management, not model ability, as the real engineering challenge
- [[Slate]] — Thread-and-episode architecture with context routing as the core primitive
- [[Loop Engineering]] — Designing systems that prompt agents rather than prompting agents yourself
- [[StrongDM Factory Techniques]] — Composable pipeline thinking applied to agent factories
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines; the end-to-end principle for AI
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy including Context and Memory
- [[The Advisor Strategy]] — Cost-aware model routing: expensive models advise, cheap models execute
- [[Thrifty (Tiered Delegation for Claude Code)]] — The same tiered-cost pattern in practice
- [[Guardrails and Feedback Loops]] — Eval, testing, and feedback as infrastructure
- [[Sawtooth Memory]] — Hierarchical memory middleware with async compression
- [[MELT]] — Benchmark harness for evaluating long-lived agent memory
- [[Components of a Coding Agent]] — The harness matters more than the model
- [[Honey I Shrunk the Coding Agent]] — Empirical proof: redesign the scaffold, 19% → 46%
- [[Context Rot]] — When agent context decays over long sessions
- [[Cerebras Knowledge Base Architecture]] — Production proof of Lee's composable-retrieval thesis: four-signal hybrid Slack retrieval with LLM distillation at write-time, serving 15K queries/day to both humans and agents via MCP
- [[New Rules of Context Engineering]] — Thariq Shihipar's official Anthropic post converges on the same prescription from the product side: progressive disclosure, delete what the model no longer needs, and treat context as composable components rather than a monolithic dump. The 80% system prompt deletion validates Lee's argument that bigger context windows are a failure of engineering imagination.

---

*Source: [AI Council interview with Linus Lee](https://www.youtube.com/watch?v=JMA9D8J9EyI), 2026-05-14. Transcript and summary via [ytx gist](https://gist.github.com/9e94fbdabe3777afd51dfea384f5c37f). Ingested 2026-07-04.*
