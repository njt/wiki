# Two Kinds of User Are Emerging

Martin Alderson identifies a widening gap between AI power users (Claude Code, MCPs, custom workflows) and casual users (basic chatbots). The surprise: many power users are non-technical professionals who learned to leverage programming ecosystems for domain-specific tasks. The enterprise barrier: locked-down environments, missing internal APIs, legacy SaaS that becomes the bottleneck rather than the enabler.

---

## Key Quotes

> "M365 Copilot has enormous enterprise market share yet feels like a poorly cloned ChatGPT interface."

> Microsoft internally adopts Claude Code despite owning OpenAI.

## Key Themes

#adoption #enterprise #power-users #api-first #legacy-systems #productivity-gap

The Microsoft Copilot observation is devastating. The company that bet $13B on OpenAI is using a competitor's tool internally because their own product isn't good enough. That's the strongest possible signal about where the real value lies: in tools that give you direct access to capabilities (Claude Code, terminal, APIs) rather than chatbot wrappers over enterprise software.

The "non-technical power users" finding challenges the assumption that AI coding tools are only for developers. A finance professional converting a 30-sheet Excel model to Python "almost one-shot" is a different kind of disruption than developer productivity. It's domain experts cutting out the middleman.

## Critical Analysis

The API-first argument is the most actionable insight for business leaders. Companies with internal APIs will be able to wire AI agents into their operations; companies without them won't. It's not about buying AI tools -- it's about having the infrastructure for AI tools to reach into. This connects directly to [[AI Killing B2B SaaS]]'s survival strategy ("become a platform") and to [[Building Agents for Production Systems with MCP]]'s technical architecture for agent-to-system connectivity.

The organic adoption observation ("employees know their processes best") is important but incomplete. Organic adoption without guardrails leads to the security nightmares described in [[AI Killing B2B SaaS]] -- finance teams storing unencrypted reports in public S3 buckets. The gap between power user capability and organizational readiness is the real tension.

---
*Sources: [[raw/two-kinds-of-user-are-emerging]]*
*Last updated: 2026-05-14*
