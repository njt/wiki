# Three Kinds of Agentic Search

Doug Turnbull disentangles the sloppy term "agentic search" into three distinct implementation strategies — retrieval-centric (build good search and let the agent use it), harness-centric (steer the agent toward relevance with judges and feedback loops), and model-centric (fine-tune an LLM to search your data efficiently) — and argues the confusion itself is a symptom of a field where nobody agrees on who's in charge: the agent, the search engine, or the model.

---

> "Agentic search gets interesting when agents don't know how to find the right answer. Agents may think they know and might confidently BS us."
>
> The core problem: agents lack domain intuition. Your users' "red shoes" means high heels; your company's "ABE" means an A/B testing tool, not a president. The agent can't know this without search — and search has to be good enough to override the agent's wrong assumptions.

> "Search leads the agent by the nose" toward relevance, overriding the agent's perspective.
>
> The retrieval-centric argument in one sentence. If your search quality is high enough, you don't need the agent to be smart about search — you just need it to ask.

> For "Synopsis of the book Ubik," the answer begins "By the year 1992, humanity has colonized the Moon and psychic powers are common." If you don't know the book, it's unclear this answers the question — the agent says "cool story bro" and ignores the info.
>
> Turnbull's sharpest concrete example. Classic RAG chunks stripped of context (titles, headings, structural cues) are invisible to agents trained on web-scale search results. The format mismatch between chunk-based retrieval and web-search-shaped expectations is a real failure mode most RAG evaluations miss.

> A judge directs the agent, correcting mistakes and guiding it toward better search strategies. Jo Kristian Bergem calls this "relevance feedback on steroids."
>
> The harness-centric insight: you don't need great search if you have a good judge. On the ESCI dataset, BM25 alone scores 0.2895 NDCG@10; adding an agentic tool-calling loop lifts it to 0.4101; one round of judge feedback pushes it to 0.5843. The judge doubles the gain the agent alone achieved.

> "Unlike a naive chunk, this bit of information has purpose" — it's clear what problem it solves when in context. This achieves what every search developer wishes content teams would do: "actually optimize content to be findable."
>
> Content optimization for agents is a different discipline than SEO for humans. A well-structured document that declares its own purpose is worth more than a perfectly-chunked but context-free one. This is the same insight behind coding-agent README files and CLAUDE.md: tell the agent *what this is for* before you tell it *what it says*.

> "The agents eat the harnesses" — when successful patterns emerge, agentic search models train to memorize them.
>
> The convergence thesis: harness patterns that work get absorbed into models via fine-tuning, just as coding-agent patterns that work get absorbed into the next generation of coding models. The harness is scaffolding; the model is the permanent artifact.

> "We're swimming in an interesting retrieval primordial goop."
>
> Turnbull's closing assessment of the field: late interaction, learned sparse retrieval, agent-centric retrieval patterns, and model fine-tuning are all evolving simultaneously. What emerges may look obvious in retrospect or nothing like any of today's components.

## Key Themes

- **#pattern** — Three-way taxonomy of agentic search: retrieval-centric (search leads), harness-centric (judge steers), model-centric (LLM internalizes)
- **#pattern** — The judge-as-oracle pattern: one round of domain-knowledge feedback doubles the NDCG gain that agentic tool-calling achieves alone
- **#concept** — Content-purpose mismatch: classic RAG chunks stripped of structural context are invisible to agents trained on web-scale search with titles, headings, and metadata
- **#concept** — Content optimization for agents is purpose-declaration, not keyword stuffing; tell the agent what the content *solves* before what it *says*
- **#concept** — The "agents eat harnesses" convergence: successful search patterns get absorbed into fine-tuned models, collapsing the three-way taxonomy into one over time
- **#tool** — Late interaction retrieval and learned sparse retrieval (LightOn, MixedBread) as the emerging retrieval primitives for agent-centric search

## Critical Analysis

Turnbull's taxonomy is useful *because* it's descriptive rather than prescriptive. He's not selling a framework; he's mapping a mess. The three categories aren't mutually exclusive — most production systems blend all three — but naming them separately forces teams to articulate which part of the problem they're actually solving. Too many "agentic search" products are really just RAG with a tool-calling wrapper, and Turnbull's ESCI benchmark numbers show exactly how much of the gain comes from each layer.

The most underappreciated insight in the piece is the content-purpose mismatch. The "cool story bro" problem — where an agent receives a perfectly relevant chunk but can't recognize it as an answer because it lacks structural framing — is a genuine failure mode that embeddings and rerankers alone can't fix. It's a content design problem masquerading as a retrieval problem. Turnbull's solution (declare the purpose of content explicitly) is obvious once stated, and almost nobody does it.

The judge-as-oracle result deserves more scrutiny than Turnbull gives it. A single round of judge feedback nearly doubles NDCG on a standard benchmark — but the judge is an oracle that knows the ground truth. In production, the judge *is* the problem you're trying to solve. The gap between oracle-judge and real-judge performance is where most harness-centric systems actually live, and Turnbull's 0.5843 NDCG number is best-case fantasy, not a production target.

The model-centric section is appropriately tentative. Fine-tuning LLMs for search-specific behavior is early-stage (SID, Glean's Waldo), and the gap between "promising results on a benchmark" and "reliable in production across diverse query distributions" is large. Turnbull's "stay tuned" is honest. But he's right that the long-term convergence pattern — harness patterns → training data → model capability — is the same one we've seen with coding agents. The model-centric approach isn't competitive today; it'll be the only approach that matters in two years.

The piece also works as an implicit argument for why [[Context Engineering at the Frontier (Linus Lee)]] is right that context engineering *is* search engineering. Every one of Turnbull's three patterns is ultimately about getting the right information into the agent's context window. The taxonomy is a lens on the same problem from the search side rather than the agent side.

## Connections

- **[[Building Reliable Agentic AI Systems]]** — Bayer's PRINCE is the most detailed public field report on production agentic RAG, and Turnbull's retrieval-centric category maps directly to its context-engineering layer. The content-purpose mismatch Turnbull identifies is exactly the kind of failure PRINCE's reflection loops catch.
- **[[Context Engineering at the Frontier (Linus Lee)]]** — Lee's thesis that composable retrieval pipelines beat monoliths is Turnbull's taxonomy from the agent side: the harness-centric approach IS composable retrieval with a judge in the loop.
- **[[Cerebras Knowledge Base Architecture]]** — A production instance of Turnbull's retrieval-centric approach: hybrid retrieval + reranker + MCP agent-native access. The four-signal Slack retrieval is the content-optimization insight applied to enterprise knowledge.
- **[[Harness Engineering is not Enough]]** — Horthy's RL critique applies directly to the model-centric approach: if you can't reward good search behavior with a fast oracle, RL-trained search models will optimize for what's measurable, not what's relevant.
- **[[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]]** — Emily Bache's Guides + Sensors flywheel is Turnbull's harness-centric approach applied to coding: feed-forward context (Guides) plus feedback signals (Sensors) steering the agent.
- **[[Context Graphs]]** — Kalra's "similarity is not relevance" is Turnbull's retrieval-centric critique from the memory side: vector similarity alone produces context-free chunks that agents can't recognize as answers.
- **[[Attemory]]** — A fourth category Turnbull's taxonomy doesn't capture: attention-native retrieval, where a local model attends over raw indexed text in its KV cache and uses forward-pass attention weights as the relevance signal. No embeddings, no similarity function, no reranker — just model attention. On SWE-QA, one Attemory hint reduced Claude Code token consumption by 43.8% with near-identical judge quality. It's retrieval-centric in that search leads, but model-centric in that the model's own attention IS the search — a fusion Turnbull's three-part taxonomy wasn't designed for.
- **[[The New Software Lifecycle]]** — Osmani's "harness over model" maps to Turnbull's argument that harness-centric approaches (judges, feedback, query plans) are where the near-term value lives, even if models eventually absorb them.
- **[[Text-to-SQL in the Real World]]** — Stonebraker's finding that enterprise data warehouses humble LLMs to 10% accuracy is the same domain-intuition problem Turnbull opens with: agents don't know your data's idiosyncrasies.
- **[[Local Deep Research]]** — A concrete instance of the taxonomy: its `langgraph-agent` strategy is harness-centric (the agent steers, engines lead it by the nose), while its encrypted library/RAG search is retrieval-centric. Its egress policy even pre-filters forbidden search engines out of the tool list *before* the LLM sees them — harness engineering of the same shape as Turnbull's judge-in-the-loop, applied to which tools may exist rather than which results are relevant.
- **[[AI Team Mistakes]]** — Turnbull's companion piece and the team-level prequel to this taxonomy: it argues AI teams are search teams, that "retrieval dictates AI quality," and that context means metadata + provenance rather than chunks — the "cool story bro" mismatch generalized from a retrieval failure into a positive RAG prescription.

---
*Sources: [[raw/three-kinds-of-agentic-search]]*
*Last updated: 2026-08-01*
