---
url: https://www.databricks.com/blog/introducing-omnigent-meta-harness-combine-control-and-share-your-agents
title: "Introducing Omnigent: A Meta-Harness to Combine, Control and Share Your Agents"
author: Matei Zaharia, Kasey Uhlenhuth, Corey Zumar
date_fetched: 2026-07-03
date_published: 2026-06-13
topics:
  - misc
---

Introducing Omnigent: A Meta-Harness to Combine, Control and Share Your Agents
Matei Zaharia, Kasey Uhlenhuth, and Corey Zumar
June 13, 2026

The article describes a problem Databricks observed: users frequently juggle 4–5 agent interfaces at once, copying text between agents and tools like Docs and Slack. Agent builders face a "treadmill" of constantly upgrading harnesses, SDKs, and models, yet these harnesses don't interoperate.

Omnigent is a "meta-harness" — a layer above existing agents (Claude Code, Codex, Pi, or custom agents) that makes them work together. It's open-sourced under Apache 2.0.

Architecture:
- **runner** wrapping any agent in a sandboxed session with a uniform API
- **server** providing policies and sharing, exposing sessions via terminal, app, and web APIs

Three pillars:
1. **Composition** — combine agents across models and harnesses with minimal code changes
2. **Control** — stateful, contextual policies enforcing guardrails (cost budgets, permissions) without relying solely on prompts
3. **Collaboration** — share live agent sessions via URL, allowing teammates to review, comment, and steer agents in real time

The core insight: regardless of how an agent harness calls its LLM, the user-facing interface is consistent — "messages and files in, text streams and tool calls out." Omnigent builds a common API wrapping terminal-based coding agents and SDKs alike.

Key features:
- Real-time collaboration: invite others to view sessions, comment on workspace files, or send commands
- Multiple interfaces: access the same agent via web, mobile, macOS app, or APIs
- Cloud execution: run agents locally or on hosted sandboxes (Modal, Daytona)
- Contextual security policies: go beyond allow/deny with dynamic state tracking. Example: after downloading an npm package, require human approval to git push
- Cost policies: pause an agent after a spending threshold (e.g., every $100) and ask to continue
- OS sandboxing: flexible sandbox that can lock down OS access and intercept/transform network requests
- Multi-harness authoring: define a custom agent in YAML and switch between harnesses with one-line changes

Roadmap:
- Automatic optimization via GEPA
- Code-based introspection within agents (referencing MemEx and RLM)
- Omnigent Server MCP so agents work across sessions
- Additional harness integrations
- Deployment support: Fly.io, Railway, Modal, Daytona, plus many LLM providers

The meta-harness vision draws a parallel to infrastructure shifts: just as engineers moved from managing individual servers to orchestration via Kubernetes and Terraform, agents need a similar abstraction lift. "Each harness is its own silo, with its own context, its own controls," and switching tools means leaving everything behind. A meta-harness makes sessions, policies, and skills portable across agents and models.
