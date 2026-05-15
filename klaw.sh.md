# klaw.sh

kubectl for AI agents: an enterprise orchestration platform distributed as a single Go binary. Provides Kubernetes-style lifecycle management (get, describe, logs), namespace-based isolation, cron scheduling, Slack integration, and multi-model routing across ~300 LLMs. Supports single-node and distributed deployments.

---

## Key Themes

#agent-orchestration #kubernetes #enterprise #scheduling #go

The kubectl metaphor is apt: if you're running agents in production, you need the same operational primitives you use for services -- list what's running, inspect state, read logs, isolate by namespace, schedule recurring work. klaw applies the Kubernetes mental model to agent management.

This is the enterprise end of the agent orchestration spectrum, opposite from [[Dorothy]] (desktop app for developers) and [[Serf]] (minimal non-interactive runner). The Slack integration (`@klaw status`, `@klaw run`) is the right channel for ops teams who live in Slack.

## Critical Analysis

Strong: the single-binary distribution and kubectl-style UX are smart choices for enterprise adoption -- ops teams already know this pattern. Namespace isolation with scoped secrets and tool permissions addresses the multi-tenant security problem. The distributed deployment model (controller + worker nodes) is necessary for scale.

Weak: source-available licensing (free for internal use, paid for SaaS) creates uncertainty about long-term availability and community contribution. The use cases described (lead scoring, competitive intelligence, support automation) are generic enough to raise the question of whether this is a real product or a landing page with a Go binary. The ~300 model support via router is just OpenAI-compatible endpoint proxying, which isn't hard.

The real test: does anyone run this in production at scale? The single-node mode is probably useful for small teams; the distributed mode needs battle-testing before you'd trust it with production agent workloads.

---
*Sources: [[raw/klaw-sh]]*
*Last updated: 2026-05-14*
