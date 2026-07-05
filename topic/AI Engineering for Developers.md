---
title: "AI Engineering for Developers"
source: "https://www.lucavallin.com/blog/ai-engineering-for-developers"
author: "Luca Cavallin"
date: 2026-06-02
ingested: 2026-06-05
tags: [ai-engineering, foundation-models, agents, rag, finetuning, inference, evaluation, prompting]
---

# AI Engineering for Developers

Luca Cavallin's field guide for backend engineers crossing into AI engineering — the discipline of building applications *on* pretrained models rather than training models themselves. Published June 2026, it's the most comprehensive single-article survey of the AI engineering landscape I've read: foundation models, prompting, evaluation, RAG, finetuning, inference optimization, agents, multi-agent protocols, and production architecture, all from the perspective of someone who has shipped.

## Precis

Cavallin opens with the observation that rewires how you think about AI work: **"The model is no longer the product. The product is the system around the model."** The model is a probability distribution, not a function — same input, different output, especially after vendor upgrades. Everything flows from this: you need eval pipelines (not unit tests), architectural defenses against injection (not prompt-level pleading), and a default posture of "workflow, not agent."

The article is structured as a layer cake. Bottom: foundation model internals (architecture, training, sampling). Middle: the three knobs for adapting an LLM — prompt engineering, RAG, finetuning — in strict cost order. Top: agents, multi-agent systems, and the protocols (MCP, A2A) that make them interoperate. Throughout, Cavallin's advice is refreshingly undogmatic: use whatever model fits the task, default to workflows over agents, build eval before shipping, and treat prompts like SQL in version control.

His prediction for the near future: "Your AI architecture in two years looks like a service mesh of agents" — specialists with focused tools, connected by standard protocols, with routing layers that dispatch cheaply before escalating to expensive reasoning models.

## Key Quotes

> "The model is no longer the product. The product is the system around the model."

> "prompt first, then RAG, then finetune. Do not skip steps."

> "Structure beats hope."

> "There is no purely-prompt-based defense against prompt injection. Architectural defenses... are the only real protection."

> "If you skip eval, you ship regressions."

> "Benchmarks lie. Models are trained on benchmarks."

> "It is not magic. It is a search engine bolted onto a generator. Most RAG bugs are search bugs."

> "Default to workflow."

> "An agent that can loop forever will eventually loop forever, and it will do it on the worst possible user request at the worst possible time."

> "A small high-quality dataset beats a large noisy one."

> "Running everything through the most expensive model is like using a rack of H100s to serve a CRUD API."

> "Treat prompts like SQL: they live in your repo, in dedicated files, versioned in git, with a CI step that runs them against an eval set."

## Themes

**1. The model is infrastructure, not product.** This is the central reframe. The model is a probability distribution you rent; the durable engineering is the system around it — eval, retrieval, guardrails, routing, caching, observability. This rhymes with [[Smart Models Dumb Pipes]] and the end-to-end principle: the model is the smart judgment layer, everything else is dumb but reliable plumbing.

**2. Don't skip steps on the adaptation ladder.** Prompt → RAG → finetune, in order of cost and reversibility. Most teams never need finetuning. The most common blocker isn't capability — it's not having an eval pipeline. This is the same discipline that shows up in [[Guardrails and Feedback Loops]]: deterministic evaluation beats hoping the prompt works.

**3. Workflows over agents, by default.** Cavallin is explicit: predefined sequences are predictable, debuggable, and cheap. Agents (where the LLM decides what to do next) are for genuinely open-ended tasks. This is the same judgment [[Agent Orchestration]] makes in its planner/worker/judge pattern — the LLM is used sparingly, at decision points, not in a tight loop.

**4. Context engineering is the real work.** Every production system enhances the user message with system prompt, metadata, retrieved docs, memory, and tool definitions — in that order. The context window is not a dumping ground. "Lost in the middle" is real. This is [[Agent Memory and Context]] territory: context management as the engineering challenge that matters more than model selection.

**5. The eval pipeline is non-negotiable.** Cavallin returns to this relentlessly. Without eval you can't iterate on prompts, can't finetune, can't detect model drift, can't compare models. The hard part isn't the scorer — it's the dataset. This connects to the broader theme in [[Guardrails and Feedback Loops]] about deterministic verification over generation.

**6. Agents need protocols, not just frameworks.** MCP standardizes tool calling; A2A standardizes agent-to-agent collaboration. Cavallin frames these as the TCP/IP of the agent era — the plumbing that lets specialists owned by different teams interoperate. This is [[Distributed Systems]] thinking applied to agents: the problems (auth, versioning, tracing, trust) are the same ones distributed systems engineering has been solving for decades.

**7. Cost-awareness as engineering discipline.** Router-based architectures with cheap SLMs classifying and dispatching to specialists. Speculative decoding for 2-3x acceleration. Prefix caching as "most of the cache hit win." Cavallin treats cost optimization not as ops afterthought but as a design constraint from day one — you don't run everything through the frontier model any more than you'd serve a CRUD API from H100s.

## Critical Analysis

**What this does well:** Cavallin has written the article I'd hand to any senior backend engineer who needs to understand the AI engineering landscape in a weekend. The scope is remarkable — from token sampling strategies to multi-agent protocols — and the advice is consistently grounded in production experience rather than demo-ware. The "three knobs" framework (prompt → RAG → finetune) alone is worth the read. The cost-awareness throughout is a corrective to the "just throw GPT-5 at it" instinct.

**What's missing or underplayed:**

*Security beyond prompt injection.* Cavallin covers injection and guardrails well, but the article doesn't engage with the broader [[Security and Sandboxing]] landscape: sandboxed execution, credential scoping, tool authorization policies. An agent with database access needs more than prompt-level defenses.

*The harness matters more than the model.* Cavallin touches on this indirectly (workflow vs agent, state management) but doesn't go where [[Honey I Shrunk the Coding Agent]] and [[Components of a Coding Agent]] go: that the scaffold around the model — how tools are presented, how state flows, how errors are fed back — can be the difference between a 19% and 46% benchmark score on the same model.

*Local and open-source inference.* The article is cloud-API-centric. The [[Local and Open Source Inference]] landscape — running Qwen, Llama, or Gemma on your own hardware — gets a nod in the model taxonomy but no real treatment. For many use cases (privacy, cost at scale, offline operation), this is the actual default.

*Agent identity and memory beyond the technical.* Cavallin covers memory types (short-term, long-term, semantic) as engineering patterns. But there's a deeper question [[Agent Identity]] raises: memory is retrieval, identity is participation. An agent that persists across sessions needs more than a vector store — it needs a stake in outcomes.

**The architecture prediction is both right and insufficient.** "Your AI architecture in two years looks like a service mesh of agents" is directionally correct but papers over the hard parts. Service meshes work because services are deterministic; agent meshes have to handle probabilistic outputs at every hop, compounding uncertainty. The [[Elements of Agentic Systems Design]] taxonomy (Context, Memory, Agency, Reasoning, Coordination) is the fuller picture — you need all ten elements, not just protocol standardization.

**Verdict:** This is the best single-article survey of AI engineering I've found. It won't replace deep dives into specific topics (read [[Building Agents for Production Systems with MCP]] for MCP details, [[Data Engineering for Large Models]] for dataset work, [[From AI Studio to AI Forge]] for the autonomy stack), but it's the map that shows you where everything fits. For backend engineers making the transition, start here.

## See Also

- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines; the end-to-end principle Cavallin's architecture implies
- [[Guardrails and Feedback Loops]] — Deterministic enforcement over prompt-level pleading; the eval discipline Cavallin insists on
- [[Agent Memory and Context]] — Context management as the real engineering challenge; Cavallin's context enhancement pipeline in detail
- [[Agent Orchestration]] — Multi-agent coordination patterns; Cavallin's router/supervisor distinction mapped to planner/worker/judge
- [[Building Agents for Production Systems with MCP]] — MCP as the standard agent-to-production integration layer
- [[Elements of Agentic Systems Design]] — The ten-element taxonomy that fills in what Cavallin's service-mesh prediction leaves out
- [[Security and Sandboxing]] — The isolation and credential-scoping layer Cavallin's article underweights
- [[Honey I Shrunk the Coding Agent]] — Empirical proof the harness matters more than the model; 19%→46% on the same 9B model
- [[Components of a Coding Agent]] — The harness taxonomy Cavallin doesn't develop
- [[Agent-Native Architectures (Every)]] — Five design principles for agent-native systems; complements Cavallin's production architecture section
- [[From AI Studio to AI Forge]] — McCormick's five-plane autonomy stack; the "human changes altitude" framing
- [[Local and Open Source Inference]] — The self-hosted landscape Cavallin nods at but doesn't explore
- [[Data Engineering for Large Models]] — Open-source textbook for the dataset engineering section
- [[OpenAI Structured Outputs]] — The structured output mode Cavallin recommends; protocol-level constraint beats prompt-level pleading
- [[KV Cache Locality]] — Prefix-aware routing for inference optimization; the infrastructure beneath Cavallin's advice to cache system prompts
- [[Agent Identity]] — Memory is retrieval, identity is participation; what Cavallin's memory taxonomy leaves out
- [[All Your Agents Are Going Async]] — HTTP is wrong for long-running agents; the transport question Cavallin doesn't address
- [[Your Coding Agent Should Do AI System Engineering]] — Burtenshaw's three-level autonomy ladder; the skill progression beyond Cavallin's overview
- [[Golem Covenant]] — Agent safety framework with default-deny; the security posture for Cavallin's service mesh
- [[Computer Use is 45x More Expensive Than Structured APIs]] — The cost gap that justifies Cavallin's router-based architectures
