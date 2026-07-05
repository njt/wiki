# Building Reliable Agentic AI Systems

Bayer AG's PRINCE (Preclinical Information Center) is the most detailed public field report on production agentic RAG in a regulated industry. Built with Thoughtworks, it evolved from keyword search to a multi-agent system that drafts regulatory documents — cutting weeks of work to minutes across 18,000+ study reports. The article's contribution isn't novelty but **thoroughness**: it names and separates two concerns that most agent builders conflate — *context engineering* (what each agent sees) and *harness engineering* (the control layer around agents). The three-reflection-loop taxonomy (process / data / draft) is worth stealing for any agent project.

---

## Key Quotes

> "production-ready agentic AI is not only about better models or better prompts. Reliability comes from engineering both the context the model sees and the harness within which the model acts."

This is the thesis and it lands. The article earns it by walking through both halves in detail rather than gesturing at them. The context/harness split is useful because they fail differently: context failures produce wrong answers, harness failures produce hung workflows. You debug them with different tools. [[Harness Engineering (OpenAI)]] makes the same point from the coding-agent side.

> "different stages receive different context"

The most important operational principle in the whole piece. Think & Plan gets planning context, Researcher gets retrieval context, Reflection gets evidence context, Writer gets synthesis context. This isn't just about token budgets — it's about preventing the model from confusing which role it's playing at each step. [[How Hightouch Built Their Long-Running Agent Harness]] converges on the same pattern independently: separated planning and execution with different context windows.

> "an agent might execute a perfectly valid workflow (good process) but still retrieve insufficient data to answer the question"

The process/data reflection distinction is the article's sharpest conceptual contribution. Most agent frameworks only have process-level feedback (did the tool call succeed?). Adding a separate data-sufficiency check — "do we actually have enough to answer?" — catches the silent failures that process checks miss. This is a [[Guardrails and Feedback Loops]] pattern applied at the retrieval layer.

> "the reviewing LLM sometimes incorrectly flagged valid queries as erroneous"

Admitting you removed an LLM-as-reviewer step because it was wrong is the kind of honesty that's rare in case studies. The meta-lesson: LLM-judge-LM is not free. Validation costs accuracy at each hop. Sometimes a SQL parser is the right tool.

> "we don't wait for features to be absolutely perfect before seeking user feedback"

Standard agile rhetoric, but worth noting in a pharma context where "move fast" usually means "get sued." The article implies Bayer's governance framework made this possible — it's not just engineering culture, it's institutional design.

---

## The Architecture Worth Stealing

### The Three Reflection Loops

| Loop | What it checks | What it catches |
|------|----------------|-----------------|
| **Process reflection** | Is the workflow on the right trajectory? | Wrong tool choice, bad sequencing |
| **Data reflection** | Is the evidence sufficient? | Thin context, coverage gaps |
| **Draft reflection** | Is the output complete? | Missing sections, synthesis gaps |

Most agent systems only have the first one. The second is the high-leverage addition — it's the difference between "the pipeline ran" and "the pipeline produced something useful."

### The 8-Step RAG Pipeline

The retrieval pipeline is overengineered by hobbyist standards and exactly right for pharma: keyword extraction → metadata filter generation → 5-way query expansion → hybrid retriever (0.7 semantic / 0.3 keyword) → cross-encoder reranking to top 7 → synthesis with citations. Every step has a clear failure mode and a monitoring hook via Langfuse. This is what production RAG looks like when "good enough" isn't.

### State Persistence as Recovery

User hits retry → system resumes from the failed node, skipping completed steps. This requires LangGraph's PostgreSQL checkpointer plus DynamoDB for application state. The engineering insight: recovery is a state management problem, not an AI problem. You can't prompt your way out of a crashed workflow.

---

## Key Themes

#agentic-rag #context-engineering #harness-engineering #multi-agent #langgraph #production-ai #pharma #reflection-loops #case-study

---

## Critical Analysis

**What's genuinely new:** The process/data/draft reflection trichotomy. It's a clean, memorable taxonomy that applies beyond this system. Every agent builder should add a data-sufficiency check — it's cheap and catches the most expensive kind of failure (confident wrong answers from thin evidence).

**What's undersold:** The article mentions "18,000+ study reports" and "weeks to minutes" but doesn't give hard numbers on accuracy, hallucination rates, or user trust metrics. For a piece that names "trust through transparency" as a core principle, the absence of quantified trust data is conspicuous. A RAGAS faithfulness score would tell us more than three paragraphs about citation UX.

**What's missing:** Cost. The article says they optimized for accuracy first and cost later, but never gives order-of-magnitude numbers. A multi-agent LangGraph workflow with 5-way query expansion, cross-encoder reranking, and reasoning-model synthesis is not cheap per query. In pharma the economics probably work (a wrong answer costs millions), but anyone copying this architecture should budget accordingly.

**The LangGraph question:** The article is effectively a LangGraph case study without saying so explicitly. The harness engineering section IS LangGraph's value proposition — stateful graphs with checkpointing, retry, and recovery. [[Apache Burr]] offers the same primitives via a different philosophy (explicit state machines over graph DAGs). The choice between them is about whether you want your workflow to be a graph or a state machine, and the article inadvertently makes the case for both.

**Comparison to coding agents:** The architecture maps surprisingly well to coding-agent patterns. Think & Plan ≈ planning phase, Researcher ≈ file reading + grep, Reflection ≈ test running, Writer ≈ code generation. The difference is that coding agents collapse process and data reflection into "did the tests pass?" — which works because code is verifiable in ways that pharma evidence isn't. This is why pharma needs the more elaborate structure.

**The NER footnote:** The named entity recognition system that auto-corrects corrupted metadata is mentioned almost as an aside, but it's arguably the most interesting engineering in the piece — a classical ML pipeline fixing the data quality problems that the fancy AI system depends on. The boring stuff enables the exciting stuff. Classic.

---

## Related Pages

- [[Harness Engineering (OpenAI)]] — Same thesis from the coding-agent side: the harness matters more than the model
- [[How Hightouch Built Their Long-Running Agent Harness]] — Independent convergence on context engineering as the real work
- [[Elements of Agentic Systems Design]] — The ten-element taxonomy this architecture instantiates
- [[Elysia]] — Alternative approach to constrained tool selection (decision tree vs. LangGraph)
- [[Apache Burr]] — State-machine alternative to LangGraph's graph-based orchestration
- [[Guardrails and Feedback Loops]] — The enforcement hierarchy: linters beat prompts
- [[Smart Models Dumb Pipes]] — Related philosophy: models own judgment, pipes own execution
- [[Agent Orchestration]] — Hub for multi-agent coordination patterns
- [[Agent Memory and Context]] — Hub for context engineering strategies

---

*Source: [Building Reliable Agentic AI Systems](https://martinfowler.com/articles/reliable-llm-bayer.html) by Sarang Sanjay Kulkarni, martinfowler.com, 2026-06-16*
