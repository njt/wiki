# Architectural Blueprints for AI Agents (Nokia Networks)

A conference talk by a senior GenAI solution architect at Nokia Networks' private-network division (name garbled in the auto-transcript; 25+ years across Microsoft, Telenor, GE) on building agents for industrial telecom systems: why they abandoned generic frameworks, why they replaced vector RAG with a knowledge-graph "agent environment," and the six-block blueprint — operation manual, structured I/O, memory, environment/context providers, toolbox, worker logic — they now build all their agents from. The most interesting talk in this wiki's collection precisely because it is about agents *inside* a live, dangerous, high-complexity system rather than agents writing code.

---

## What It Argues

The business case is scarcity: Nokia's private-network division (2,000 people) serves mines, trains, and energy, where one network function has "a configuration file that contains 27,000 configuration entries." Agents exist to make scarce engineers productive, then to run autonomously inside the network itself.

Two learnings frame everything. Generic frameworks don't survive industry — they tried LangChain, LlamaIndex, and Atomic Agents, found "totally different abstractions," and concluded you must build your own hosting, orchestration, and architecture. And vector RAG "simply not enough": thousands of PDFs, Confluence pages, and JIRA tickets poured into vector DBs delivered similarity, not relationships. The fix is a deeply connected property graph fed by four pipelines — DITA XML documentation ("we try to go to the roots," not PDFs), a live Kubernetes-cluster scan that "mirrors the real world to the agents," functional requirements, and all failures/incidents.

His definition of an agent is deliberately unromantic: "a kind of semantic execution logic" — a class with structured input and output, "but in a semantic way," necessarily mixing LLM calls with native "worker logic." The six-block blueprint wraps that logic: operation manual (identity, purpose/expectations, ontology), structured I/O, memory, environment/context providers, toolbox, worker logic. The environment is the axis he insists everyone else skips — agents don't just read the graph, they write back, "creating new knowledge and... evolving the graph."

## Key Quotes

> "Simply we don't have enough engineers to manage, plan, operate, fix, support the networks."

The entire business case in one line. Note what it is not: not novelty, not a demo, not "AI transformation." A labor shortage, with a 27,000-entry config file as the scale of the problem.

> "We just realized that for industry strengths, really advanced agents, somehow we need to avoid generic frameworks."

The verdict after a year with LangChain, LlamaIndex, and Atomic Agents. One team's experience, not a law — but from the part of the industry where agents must be embedded, not appended.

> "This semantic similarity method, which is called this RAG method, simply not enough."

The talk's most contrarian claim, delivered without a benchmark. The obvious counterargument — that their chunking/metadata/retrieval pipeline was the problem, not the method — is never entertained.

> "It's a class that takes a structured input, it is doing some operations, it is generating a structured output, but in a semantic way."

His entire definition: boring object-orientation plus judgment. Refreshing against the mystical framing of agents elsewhere; the class metaphor keeps the host/orchestration side tractable.

> "When you build an agent, think about the environment, how it is represented, how the agent is connected to the environment, and how you get context, but also how you give back information to the environment."

The design principle he most wants to land. The second half — agents *writing back*, evolving the graph — is the part most agent designs omit entirely.

> "An agent can build another agent."

Dropped almost casually while explaining the repository pattern for operation manuals: stored externally, reloaded at runtime, therefore modifiable by another agent. The most radical line in the talk, and it is never elaborated — no governance, no versioning, no safety.

> "The problem is that LLMs are really bad at structured outputs."

Why every agent I/O goes through the Instructor library and Pydantic contracts — which is also how chained agents stay compatible (one agent's output dataclass is the next agent's input).

> "I discourage to just take a REST API and just mimic the API as a tool for an agent."

A direct hit on the most common shortcut in agent tooling: tools must be *described* — what they do, inputs, outputs — so the agent can reason about when to use them.

> "We cannot build all the tools that we need. Maybe someone else will build the tools."

The honest rationale for betting on MCP: not ideology, arithmetic.

> "Sometimes it's good, sometimes it's bad."

His blunt assessment of LLM-summary memory — the middle rung of his four-rung ladder (queue → summaries → vectorized → graph). Un-marketed and better for it.

## Key Themes

#concept — the agent as "semantic execution logic"; the **agent environment** as a first-class design axis; the **ontology** in the operation manual ("a schema for the world"); the four-rung **memory ladder**; the six-block blueprint.

#tool — Neo4j and embeddable KùzuDB (in-memory analytical graph engine), Cypher, Instructor + Pydantic for structured I/O, DITA XML for source-of-truth documentation, MCP for external tool discovery.

#pattern — router + handoff, parallel + synthesizer, evaluator/observer loops (all credited to Anthropic's blog); repository pattern for externally-stored operation manuals; command architecture for the toolbox; vector-search-seeded graph traversal as the Graph RAG retrieval method.

#person — the speaker, a Nokia Networks GenAI solution architect (auto-transcript mangles his name; ex-Microsoft, Telenor, GE).

## Opinionated Take

The environment-first thesis is the real contribution. Most of this wiki's agent literature is model-centric — prompts, loops, harnesses. This talk is systems-integration-centric: the graph *is* the product, the four injection pipelines are the engineering, and the LLM is just the reasoning layer over it. That inversion is what "agents embedded in real systems" actually looks like, and his insistence that agents write back into the environment is genuinely rare — most designs treat context as read-only.

But confidence outruns evidence throughout. There is not one number in the talk: no time saved, no tickets resolved, no before/after — in a presentation whose premise is engineer productivity. The "RAG simply not enough" claim is asserted after describing a pipeline failure, without entertaining that the pipeline was the problem. Graph drift is waved at ("mirror the real world") with no answer for how often the K8s scan runs or who maintains the ontologies — presumably the same scarce domain experts the agents are meant to multiply, which the Q&A quietly concedes ("you will need to talk to domain experts").

The loudest omission is safety. A troubleshooting agent that "runs commands" and writes/executes bash and Python on live network functions — with no guardrails, approval gates, rollback, or blast-radius discussion — is presented as the flagship scenario, including five-minute network-slice provisioning where nobody asks whether an LLM loop can even hit that latency SLA. For a division whose product is network reliability, the silence is remarkable. Similarly thin: evaluation ("an Excel file and an evaluator agent... a quality score") with no regression testing, no production monitoring, no handling of behavior drift across model updates — and no answer to who evaluates the evaluator.

Worth flagging: he calls Cypher "the international graph query language"; it is Neo4j's language (the ISO standard is GQL, which Cypher influenced). Minor, but consistent with the talk's pattern of confident claims resting on less than they sound like. And a factual wobble in the demo premise: the transcript's "Formula One event... 100 VIP person" versus "500 people" minutes later suggests the auto-transcript itself is lossy in places.

The framework-abandonment verdict deserves a sympathetic reading, though. It is not "frameworks bad" — it is that generic abstractions are built for the median use case, and industrial agents are not the median. That is the same conclusion every serious harness-builder in this wiki has reached from a different direction.

## Related Pages

- [[Context Graphs]] — strengthens it with vocabulary: Karan Kalra's "similarity is not relevance" is exactly Nokia's claim, and Nokia's talk shows what the industrial-scale version looks like — four injection pipelines and agent write-back that evolves the graph, which the context-graph essay only gestures at.
- [[Your Knowledge Graph Is Making Your Agent Dumber]] — the direct counterpoint. Praveen Vijayan's empirical takedown of code knowledge graphs vs. Nokia's enthusiasm for industrial ones; both can be right, because a telecom system is graph-shaped by nature (services, interfaces, dependencies) while a codebase mostly isn't. The interesting question the pair poses: when is the world actually graph-shaped?
- [[Vedana — Domain Models as Agent Context]] — the closest sibling: Epoch8's hand-built domain models, graph database, and Cypher queries as agent context, with domain experts as the maintenance bottleneck. Nokia industrializes the same bet with automated pipelines; both talks end at the same human wall.
- [[OpenAI Structured Outputs]] — nuances the structured-I/O block: the speaker's "LLMs are really bad at structured outputs" is the pragmatist's answer (Instructor + Pydantic enforcement); OpenAI's protocol-level schema guarantee is the platform answer. Same problem, two layers of the stack.

---
*Sources: [[raw/architectural-blueprints-for-ai-agents]], [[summary/architectural-blueprints-for-ai-agents]]*
*Last updated: 2026-09-13*
