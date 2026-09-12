# AI Agents In-Depth — Function Calling, MCP and Tool Use Under the Hood

Alan Smith's demo-heavy NDC talk is a practitioner's corrective to the magical thinking around AI agents: the LLM doesn't call tools, the agentic application does. Through live demos — raw JSON in Postman, a pizza-ordering agent, a vibe-coded website builder, a Wikipedia RAG pipeline, and a DJ MCP server — he surfaces a catalog of failure modes that every agent builder should internalize. The talk also delivers the clearest public clarification of what MCP is and isn't, and why the distinction matters.

---

## Key Quotes

> "The large language model doesn't call a tool. What the large language model does is selects the tool and how the tool should be called."

This is the talk's foundational distinction, and it's worth tattooing on every agent builder's monitor. The LLM returns a function name and parameters; the agentic application makes the actual call, passes the result back, and the LLM composes a response. The LLM is a decision engine, not an execution engine. This maps directly to the [[Smart Models Dumb Pipes]] thesis — LLMs as judgment machines, not Q&A machines — and explains why the harness around the model matters more than the model itself. Every framework, SDK, and protocol in this space is ultimately plumbing around this single architectural fact.

> "I've seen blog posts and articles and YouTube videos telling you that the model context protocol does what's shown on this slide, but it does not."

Smith draws a sharp line that many conflate: the "model context" is the JSON payload you send to an LLM (system prompts, conversation history, tool definitions, tool responses). The Model Context Protocol is a protocol for distributed tool servers. They're completely different things. The model context format varies by provider (OpenAI, Anthropic, Gemini all do it differently); MCP is about how agents discover and communicate with remote tools. This is the single most useful clarification in the talk, and it's one that even authoritative-sounding YouTube explainers get wrong.

> "If your only tool is a hammer, you see every problem as a nail. But LLMs... tend to think that if they've got a tool, it can basically do what the user is requesting and it's very often incorrect."

The hammer-nail problem, demonstrated with a memorable failure: asked to generate a QR code for a URL, the LLM used DALL-E (an image generator) rather than a QR code library. DALL-E produced something that *looked* like a QR code but wasn't scannable. The LLM doesn't understand *how* its tools work — it only knows their descriptions. This is a category error that description quality alone can't fix; the model needs to understand tool *capabilities*, not just tool *descriptions*.

> "If unintentionally or maliciously tools could start providing wrong answers, then it opens up all kinds of bugs... the LLM will just believe the results of the tool."

Demonstrated with a calculator tool returning `a + b + 1`. The LLM accepted the wrong answer until Smith challenged it with parity reasoning ("if you add two evens together, you get an even number"). The LLMs are trained to trust tool outputs, and this trust is naive — there's no built-in skepticism or cross-validation. This is a structural vulnerability that [[Building Reliable Agentic AI Systems]] addresses with its three-reflection-loop taxonomy, but Smith's demo makes the failure visceral rather than theoretical.

> "It's just sent a credit card number without batting an eyelid."

In the pizza-ordering demo, the system prompt contained a credit card number and a tool accepted a credit card parameter. The LLM passed the number through without hesitation. This is the data-leakage vector that keeps security engineers up at night: LLMs treat all available information as fair game for tool parameters. When Smith switched from OpenAI to Azure Foundry, the behaviour changed — the model sometimes refused — but the non-determinism means you can't rely on refusal. [[How We Contain Claude]] documents the same pattern from the other side: permission prompts get a 93% approval rate, and users are the injection vector.

> "The models are non deterministic, so if we give it the same statement, it's probably going to do something different and make different orders."

Same conference-pizza prompt, different pizza orders every run. Non-determinism isn't just a testing problem; it's a correctness problem. You can't write assertions against a system that makes different choices each time. Smith notes that behaviour also changes over time as providers update models — something that worked last week might not work this week. This is the testing crisis that [[A New Era for Software Testing]] and [[Agentic Testing]] are responding to, and the fundamental reason [[Eval-Driven Development (Airbnb)]] treats evals as infrastructure, not afterthoughts.

> "MCP is cool. I recommend looking into it, but do you really need it? If you can build something simple with function calling, you don't need the additional complexity of working with MCPs."

The pragmatic counterpoint to MCP enthusiasm. Smith frames the decision as an architectural one — function call vs. microservice — rather than a capability one. MCP adds the complexity of a distributed system (streaming HTTP, notifications, server management) and you should only pay that cost when you need the benefits (distribution, shared tool servers, long-running task notifications). This aligns with [[Stateless MCP]] finding that stateless HTTP removes the deployment tax that made MCP unnecessarily complex for simple cases.

> "I prefer Semantic Kernel. It was more C Sharpy than Agent Framework. I think Agent Framework is developed by Python developers, so it's more Pythonic."

A rare practitioner's comparison of the two Microsoft agent frameworks. Smith's preference for Semantic Kernel is aesthetic ("more C Sharpy") but the practical insight is that Agent Framework's cross-language consistency (similar code in Python and C#) matters less than framework ergonomics for your primary language. [[Semantic Kernel]] covers the architectural tradeoffs; Smith adds the developer-experience dimension.

## Key Themes

#agent-architecture #tool-calling #mcp #function-calling #security #non-determinism #vibe-coding #rag #context-management #multi-agent #framework-comparison #failure-modes

## Critical Analysis

**The "LLM doesn't call tools" distinction is the most important sentence in the talk, and it's still underappreciated.** The industry routinely talks about "agents calling tools" as if the LLM reaches out and executes code. It doesn't. This isn't pedantry — it's the architectural fact that determines where security boundaries go, where failure handling lives, and why the harness is the hard part. Every agent builder who hasn't internalized this will eventually debug a production incident where they assumed the model was doing something it structurally cannot do.

**The MCP clarification is public service.** The Model Context Protocol has been widely misunderstood — even by people writing explainers and tutorials — as a universal format for communicating with LLMs. It's not. The "model context" (the JSON you send) is handled by provider SDKs. MCP is a protocol for distributed tool servers. Smith's slide showing the "which this is not" diagram is worth more than a dozen blog posts. If the MCP community adopted clearer naming, a lot of confusion would evaporate. [[Bringing MCP 2026-07-28 to Claude]] and [[Stateless MCP]] cover where MCP is going; Smith covers what it actually is.

**The failure mode catalog is the real value, not the architecture walkthrough.** The Postman-level JSON demo is fine, but the demos that break are the ones that teach. The hammer-nail problem (wrong tool chosen), tool trust (wrong answer accepted), credit card leakage (data passed without judgment), and non-determinism (different orders every run) form a tight catalog of agent failure modes. Each is demonstrated live, not described in the abstract. This is the talk's pedagogical strength: Smith shows you the thing breaking, then explains why.

**The non-determinism problem is worse than Smith lets on.** He demonstrates that the same prompt produces different tool calls, but the deeper problem is that provider-side model updates change behaviour silently. A system that passes security review today might fail tomorrow because OpenAI or Anthropic updated the model. This isn't a bug; it's a property of the deployment model. The only honest answer is continuous evaluation — which is what [[Eval-Driven Development (Airbnb)]] prescribes — but Smith doesn't go there. He flags the problem; the solution lives elsewhere in the wiki.

**Smith is refreshingly framework-agnostic in a space full of partisans.** He shows the same pizza agent in three frameworks (LangChain, Semantic Kernel, Agent Framework), builds MCP demos in both Python and C#, and his conclusion is pragmatic: use what works for your language and context, don't add MCP unless you need distribution. The preference for Semantic Kernel over Agent Framework is a matter of taste ("more C Sharpy"), not a religious war. This is the right tone for a field where framework churn is constant and yesterday's best practice is tomorrow's deprecated API.

**The security section raises problems without solutions.** Smith demonstrates credit card leakage, Copilot Studio's anonymous-access danger, and non-idempotent database modifications — but doesn't offer mitigations. The omission is honest (these are hard problems) but the talk would be stronger with even a sketch of defensive patterns: output validation, sandboxing, human-in-the-loop for sensitive operations, least-privilege tool access. [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and [[How We Contain Claude]] fill these gaps from different angles. [[Corsair — Agent Integration Layer]] is the concrete implementation of the sketch: it wraps each integration as a typed plugin with per-endpoint risk metadata, so least-privilege tool access and human-in-the-loop-for-destructive-actions come for free at the harness's tool-call boundary — the exact layer Smith identifies as where the call actually happens.

**What's missing: cost, evals, and the production story.** Smith covers the what and how of tool calling, but not the production concerns: token costs, evaluation strategies, monitoring, error handling when tools fail, or how to version tools. The talk is a deep dive into mechanism, not operations. For production readiness, pair it with [[Building an Advanced Agentic Harness]] (the primitives you need beyond the basic loop), [[Cerebras Knowledge Base Architecture]] (production RAG at scale), and [[The MCP Gateway Iceberg]] (what breaks when MCP hits production).

---

*Sources: [[raw/98f0d215f58d9356776deb03ee41e044]], [[summary/98f0d215f58d9356776deb03ee41e044]]*
*Last updated: 2026-08-07*
