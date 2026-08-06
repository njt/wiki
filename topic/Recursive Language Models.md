# Recursive Language Models

RLMs are an inference strategy that wraps a language model in a REPL environment, letting it recursively call itself or other LLMs to process context of unbounded length. Instead of stuffing everything into one context window — and suffering the "context rot" that degrades model performance as context grows — an RLM gives the root LM programmatic access to the context as a Python variable, and lets it decide at test time how to peek, grep, chunk, and delegate. The bet: LMs should decide how to break down context for LMs, not humans designing fixed retrieval pipelines.

---

## Key Quotes

> "There is this well-known but difficult to characterize phenomenon in language models known as 'context rot'... it's almost like, as the conversation goes on, the model gets…dumber? It's sort of this well-known but hard to describe failure mode that we don't talk about in our papers because we can't benchmark it."

Zhang names the problem that every heavy Claude Code user knows but struggles to articulate. This is the same phenomenon described in [[Tuning Claude Code Into a Better Engineering Partner]] — re-suggesting rejected ideas, forgetting context already discussed — but Zhang's contribution is framing it as a *structural* problem solvable at the inference layer rather than a session-management discipline.

> "We take a context-centric view rather than a problem-centric view of input decomposition."

The single sentence that distinguishes RLMs from agents. Agents (ReAct, ROMA, multi-agent orchestrators) decompose by *task* — break the problem into sub-problems, assign to sub-agents. RLMs decompose by *context* — break the data into chunks, query each, synthesize. The distinction is subtle but the implications are large: an RLM doesn't need a human to pre-structure the decomposition strategy. The REPL environment is the interface, and the LM figures out the rest.

> "We demonstrate that an RLM using GPT-5-mini outperforms GPT-5 on a split of the most difficult long-context benchmark we got our hands on (OOLONG), double the number of correct answers, and is cheaper per query on average."

The empirical headline. GPT-5-mini through an RLM wrapper beats raw GPT-5. This is the same pattern as [[Honey I Shrunk the Coding Agent]] — the harness matters more than the model — but applied to long-context reasoning rather than coding. A weaker model with a smarter scaffold beats a stronger model running solo.

> "RLMs are a fundamentally different bet than modern agents. Agents are designed based on human / expert intuition on how to break down a problem to be digestible for an LM. RLMs are designed based on the principle that fundamentally, LMs should decide how to break down a problem to be digestible for an LM."

The philosophical thesis of the paper. It's a bet on model capability over human design — the same bet that made chain-of-thought reasoning work (let the model figure out the steps) rather than scripted decomposition. Whether it pans out depends on whether LMs get good enough at recursive self-orchestration before the alternative (bigger context windows) makes the whole approach unnecessary.

## Key Themes

- #concept **Context-centric decomposition** — RLMs decompose by context (the data), not by problem (the task). The root LM sees only the query; the full context lives in a REPL variable it can programmatically access.
- #pattern **REPL as environment** — A Python/Jupyter-style notebook pre-loaded with the context as a variable. The LM writes code to peek, grep, partition, and spawn recursive sub-calls. Code execution is the mediation layer between the LM and its context.
- #concept **Inference-time scaling, third axis** — After chain-of-thought reasoning (let the model think longer) and ReAct agents (let the model use tools), RLMs add recursive context decomposition (let the model decide how to read). Each is learnable and RL-ifiable.
- #pattern **Emergent strategies** — RLMs spontaneously develop peeking (sampling the first N chars), grepping (regex to narrow context), partition+map (chunk and delegate), and summarization. These aren't pre-programmed; they emerge from the LM interacting with the REPL.
- #concept **Context rot as a solvable problem** — Zhang argues we don't need to *fix* context rot at the architecture or training level if no single LM call ever handles a huge context. Decompose before the rot sets in.
- #tool **OOLONG and BrowseComp-Plus** — The two benchmarks that actually stress long-context reasoning, unlike popular benchmarks where models can answer without reading the context.

## Critical Analysis

**What's compelling:** The results are genuinely surprising. A GPT-5-mini wrapped in an RLM outperforming raw GPT-5 on long-context tasks is the kind of result that should make architecture teams reconsider whether bigger context windows are the right investment. The REPL-as-environment choice is clever — it piggybacks on models' existing code-writing ability rather than requiring new infrastructure. And the emergent strategies (peeking, grepping, chunking) are the kind of spontaneously-arising behavior that suggests the approach has room to grow as models improve.

The positioning as a third axis of inference-time scaling is the right ambition. Chain-of-thought gave us reasoning models. ReAct gave us agent models. If RLMs give us models that can recursively manage unbounded context, that's a genuine new capability class — and unlike agent frameworks, the decomposition strategy is learned, not designed.

**What's concerning:** The results are on small subsets (20 queries on BrowseComp-Plus) and the paper isn't peer-reviewed. The OOLONG benchmark authors shared their dataset upon request, which is great for openness but means there's no independent reproduction yet. The speed caveat is significant — each recursive call is blocking and sequential, so an RLM query can take "several minutes." For interactive use cases (Claude Code, ChatGPT), that's a non-starter without async execution and prefix caching, both of which Zhang acknowledges as "low hanging fruit."

The recursive depth of 1 is both a strength (it works!) and a limitation (we don't know if deeper recursion compounds or collapses). Zhang is upfront about this, but the architecture's name promises more than the current implementation delivers.

**Compared to existing approaches:** RLMs sit in an interesting position in the context-management landscape. [[Context Engineering at the Frontier (Linus Lee)]] argues for composable retrieval pipelines designed by humans. RLMs argue the LM should design the pipeline at test time. [[So Long and Thanks for All the Context]] prescribes five manual techniques (curate, position, restart, restate, verify). RLMs automate the decomposition but require a REPL environment. [[New Rules of Context Engineering]] is about reducing what goes into context. RLMs are about keeping context out of the LM entirely until needed.

The closest existing pattern is [[How Hightouch Built Their Long-Running Agent Harness]]'s subagent fanout — spawn isolated workers, return only summaries. RLMs generalize this: instead of a fixed fanout pattern, the LM decides the fanout strategy per query. And unlike [[The Advisor Strategy]] or [[Thrifty (Tiered Delegation for Claude Code)]], which are about model-tier routing, RLMs are about recursive self-delegation at the context level.

**Who should care:** Anyone building systems where context length is the binding constraint — long-running agent sessions, document QA over large corpora, multi-hop reasoning across many sources. If you're paying for context windows and watching performance degrade as they fill up, RLMs are a bet worth tracking. If your context fits comfortably in a single window and performance is fine, this is academically interesting but not immediately actionable.

**The open question:** Will RLMs be obsoleted by bigger context windows before they mature? Zhang's answer is that RLMs scale *with* context window improvements — if tomorrow's models handle 10M tokens, RLMs handle 100M. But that assumes the decomposition overhead stays constant, which it won't if models get better at long-context reasoning and the RLM wrapper becomes unnecessary overhead. The race between architectural solutions ([[Recent Developments in LLM Architectures]]) and inference-strategy solutions (RLMs) is the real story here.

---

*Sources: [[raw/rlm]], [[summary/rlm]]*
*Last updated: 2026-08-06*
