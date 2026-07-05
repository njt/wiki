# LLMs Are Complicated Now

Ian Barber's sharp diagnosis of how LLM architectures have evolved from clean, uniform transformer stacks into baroque systems that increasingly resemble the "terrifying" complexity graphs of production recommendation systems — and why composability, not agentic cleverness, is the only way out.

---

> "Attention might be all you need, but it turns out there are many flavours of attention, and now they're all in the mix."

The early Llama-era models were remarkably uniform: repeatable blocks of self-attention followed by feed-forward layers. That era is over. Barber catalogues the proliferation: query grouping, compressed attention, sparse attention, linear attention, sliding-window attention — all deployed simultaneously in different parts of the same model. Mixture-of-Experts, once confined to feed-forward layers, now routes attention blocks and even residual streams. Vision and audio encoders are tightly integrated rather than bolted on. Multi-GPU inference introduces communication ops that create additional structural boundaries. The architectural graph of a modern LLM looks less like a clean stack and more like the recsys diagrams that ML engineers used to joke about.

> "That tension between the need to continually increase capabilities and the need to stay efficient is driving more and more complexity in LLM architectures, just as it did in recommender systems."

## The Recsys Parallel

Recommendation systems stayed architecturally simple — essentially two-tower sparse neural nets — for years. Then the capability-efficiency tension drove them into baroque complexity. Barber argues LLMs are on exactly the same trajectory, and for exactly the same reason. This is the article's most original move: recasting LLM architecture evolution not as a story of scientific breakthroughs but as an engineering response to the same optimization pressure that produced the "terrifying" recsys graphs.

> "You can't generate your way forward without a baseline to check."

## The Optimization Trap

The article's sharpest argument is that agentic approaches — having LLMs themselves design better architectures — can't solve this. Without efficient fused implementations of novel attention variants, you can't even evaluate whether they're worth pursuing. A 10% slowdown is tolerable in research; an order-of-magnitude slowdown makes a variant unresearchable. But building those efficient implementations for every candidate variant is prohibitive. The result: the search space of possible architectures is artificially constrained by implementation cost, not by merit.

Barber's conclusion is blunt: **"the only way out is to design for composability up front."**

> "Being able to cut architectures to their essence and make them composable will matter at least as much as clever auto-research loops set up by agents."

## FlexAttention as Existence Proof

PyTorch's FlexAttention is held up as the model: it generated Triton kernels for an entire class of attention operations, enabling exploration "with only a very mild impact to performance." The key design property was composability — building blocks that could be verified independently and combined without exponential complexity explosion. This is the positive counterexample to the recsys doom-spiral.

## On Karpathy and Anthropic

Barber notes Karpathy joined Anthropic to advance auto-research loops, but gently argues that architecture-first composability work matters more than agentic research setups. The implicit question: is Anthropic betting on the right lever?

---

## Critical Analysis

**The Jevons Paradox of architecture.** Barber's argument implies a nasty dynamic: efficiency improvements (like FlexAttention) enable more architectural exploration, which produces more complexity, which demands more efficiency improvements. The treadmill never stops. He doesn't explicitly name this as a Jevons paradox, but it's the structural dynamic he's describing — and it means composability isn't a one-time fix, it's an ongoing arms race against your own success.

**The agentic skepticism is well-aimed but underspecified.** "You can't generate your way forward without a baseline" is a good line, but it conflates two different claims: (1) agents can't invent novel kernels without fused implementations to test against, and (2) agents can't make architectural decisions at all. Claim 1 is clearly true. Claim 2 is less obvious — agents might be perfectly capable of architectural reasoning given the right abstractions. The real question is whether agents can reason about *composability properties*, which is a different skill than kernel generation.

**The recsys analogy is a warning, not just an observation.** Recommendation systems didn't just get complicated — they became unmaintainable, inscrutable, and fragile. If LLMs follow the same trajectory, we're heading toward models whose internal behavior nobody fully understands, debugged through superstition and A/B testing. Barber is too polite to say this explicitly, but it's the implication.

**What's missing: the organizational dimension.** Architectural complexity isn't just a technical problem — it's an organizational one. Recsys complexity was driven partly by ML teams optimizing local metrics without global coordination. The same dynamic is visible in LLM development: attention teams, MoE teams, multimodal teams, and inference teams each adding their own complexity budget. Composability is a technical answer to what is partly an organizational problem.

**Karpathy's auto-research loops vs. Barber's composability is a false binary worth taking seriously.** Both are necessary. Auto-research loops without composable architecture will drown in implementation cost; composable architecture without auto-research will be under-explored. The synthesis is probably: composable architecture as the substrate, auto-research loops as the exploration strategy. Barber is right that composability is currently underinvested relative to agentic approaches, but the strongest position is "both, and composability first."

---

## Key Themes

- #concept — Architectural complexity growth as a response to the capability-efficiency tension
- #pattern — Composability as the defense against unmaintainable complexity
- #tool — FlexAttention and Triton as existence proofs for composable kernel design
- #comparison — Recsys evolution as a cautionary parallel for LLM architecture
- #concept — The optimization trap: implementation cost constraining the search space

---

*Sources: [[raw/llms-are-complicated-now]]*
*Last updated: 2026-07-05*
