---
url: https://stripe.dev/blog/meet-stripes-knowledge-ai-platform
title: "Meet Stripe's Knowledge AI Platform"
author: Anna Mason, Sharadh Krishnamurthy, Anupam Upadhyay
date_fetched: 2026-09-25
date_published: 2026-07-30
topics:
  - agent-architecture
  - agent-orchestration
---

Stripe's engineering blog introducing Kai, the internal knowledge AI platform that brought agentic leverage to non-engineers (sales, finance, marketing, TAMs) after coding agents had transformed only engineering. Within two weeks of the April launch most of Stripe used it; 83% are weekly active users, including nearly all of go-to-market.

The argument: knowledge work is the opposite of coding from an agent-architecture standpoint — no uniform tool loop, no compiler, no tests, no git to catch mistakes. Three problems had to be solved: scaling expertise without centralizing it (domain knowledge lives across dozens of orgs, not in the platform team), meeting users wherever they work (a platform, not an app — web, Slack, Chrome extensions, embedded APIs), and enforcing guardrails that don't exist in code, like task-scoped data isolation between customer contexts.

Architecturally, Kai is three layers: surface-agnostic APIs (the agent is a service, not an application), Agent Studio as the control plane where domain owners build and govern their own agents and skills, and a shared execution environment — harness built on LangChain's deepagents, Kubernetes with per-session sandboxes, a multi-tenant virtual filesystem, deliberately shared with Stripe's product-facing agents so security bar and improvements propagate to both. The harness handles 1,000+ skills/tools with a hybrid RAG/LLM skill-selection approach, and sessions reach 932 turns.

Impact numbers are striking: new GTM hires are "Kai-native" (2.7x more usage), AEs produce 2x sales activity and close 39% more deals in weeks they use it, ~25,000 hours/year shifted from admin to revenue work, 5,000+ sessions/day on data analysis. Open questions: better state management across active vs extended context, a reflection/self-improvement loop for skills, and collaboration primitives so session context isn't locked in.
