# Semantic Kernel

Microsoft's open-source SDK for building AI agents and integrating LLMs into C#, Python, and Java applications. Positions itself as "middleware" -- the translation layer between model function-call requests and your existing code. Used internally at Microsoft and by Fortune 500 companies.

---

## Key Quotes

> "Semantic Kernel combines prompts with existing APIs to perform actions."

> "When a request is made the model calls a function, and Semantic Kernel is the middleware translating the model's request to a function call and passes the results back to the model."

## Key Themes

#agent-architecture #tool #microsoft #sdk #middleware #enterprise #plugin

**Function calling as the core primitive.** Semantic Kernel's central design bet is that you describe your existing code to the model, and the model decides when to call it. The SDK handles the plumbing: serializing requests, routing to the right function, passing results back. This is the same pattern that [[Building Agents for Production Systems with MCP]] describes as the MCP approach, but implemented as an in-process SDK rather than a protocol.

**Plugin architecture via OpenAPI.** Existing code becomes a "plugin" that the agent can invoke. The use of OpenAPI specifications for plugin definitions means extensions are shareable across Microsoft's ecosystem -- the same plugin works in Semantic Kernel and in Microsoft 365 Copilot. This is the "group tools around intent, not endpoints" principle from the MCP guide, but using a different standard.

**Enterprise-first positioning.** Telemetry, hooks, filters, multi-language support (C#, Python, Java), and a commitment to non-breaking changes after v1.0. This is Microsoft doing what Microsoft does: making the enterprise-grade plumbing that individual developers won't build themselves.

**Model-agnostic by design.** Swap models without rewriting code. The abstraction layer sits between your business logic and whatever LLM you're using, so model upgrades are configuration changes, not refactors.

## Critical Analysis

Semantic Kernel is the enterprise establishment's answer to the agent framework question, and it shows. The overview page is almost entirely marketing language -- "enterprise-grade," "future proof," "rapid delivery" -- with very little technical substance. Compare this to the [[Components of a Coding Agent]] decomposition (six concrete architectural components) or [[Elements of Agentic Systems Design]] (ten behavioral elements). Semantic Kernel's overview tells you what it wants to be, not how it works.

The interesting tension is between Semantic Kernel and MCP. Both solve the same problem: connecting LLMs to existing code and APIs. Semantic Kernel does it as an in-process SDK with OpenAPI plugins. MCP does it as a protocol with remote servers. Microsoft ships both -- [[DAB]] already has an MCP server. The bet seems to be that Semantic Kernel wins inside the enterprise firewall (tight integration, telemetry, compliance) while MCP wins at the boundary (cross-vendor, cloud-hosted agents, composability). Whether these converge or compete is an open question.

The C#/Python/Java language support is notable for what it excludes: no JavaScript/TypeScript, no Go, no Rust. This is a bet on enterprise backend languages, not on the polyglot agent ecosystem this wiki mostly tracks. For the Claude Code / coding agent world, Semantic Kernel is largely irrelevant -- it solves enterprise integration problems, not developer workflow problems.

The page is thin on what actually matters: memory architecture, context management, evaluation, orchestration patterns. The [[Elements of Agentic Systems Design]] framework covers ten elements; Semantic Kernel's overview addresses maybe two (Agency and Context). For a serious evaluation, you'd need to go deeper into their docs on planners, memory connectors, and the kernel's execution pipeline.

**Bottom line:** Semantic Kernel is Microsoft's bet that the agent middleware layer will be an SDK, not a protocol. It's well-positioned for C#/.NET enterprise shops that need to add AI capabilities to existing codebases. For everyone else, MCP and lighter-weight approaches are more likely to win.

## Cross-links

- [[Building Agents for Production Systems with MCP]] -- the protocol-based alternative to SDK-based agent integration
- [[DAB]] -- Microsoft's other agent-to-data bridge, which ships both OpenAPI and MCP
- [[Components of a Coding Agent]] -- what a serious agent architecture decomposition looks like vs. this marketing overview
- [[Elements of Agentic Systems Design]] -- ten-element taxonomy that exposes what Semantic Kernel's overview leaves out
- [[Agency]] -- composable agents from natural-language primitives, the MCP-native alternative
- [[markitdown]] -- another Microsoft open-source AI tool, document conversion for LLM pipelines
- [[Spec-First Development at Benchling]] -- "define once, consume everywhere" philosophy that parallels Semantic Kernel's plugin model
- [[Agent Design & Architecture]] -- synthesis page for the agent framework landscape

---
*Sources: [[summary/semantic-kernel-overview]]*
*Last updated: 2026-05-14*
