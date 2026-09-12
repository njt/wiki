---
title: "Loomkin"
url: https://github.com/pass-agent/loomkin
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-orchestration
---

Multi-agent platform built on Erlang/OTP (actually Elixir). Agents form teams, spawn specialists in milliseconds, share discoveries in real-time, review each other's work, debate approaches and vote on decisions, self-heal when things break, verify output before moving on.

Key differentiators from traditional AI assistants:
- Teams-first: every session is a team of 1+ that auto-scales
- Decision graph: persistent DAG of goals, decisions, outcomes (7 node types, typed edges, confidence scores) -- not just chat history
- Context Mesh: overflow offloaded to Keeper processes with staleness tracking and failure memory, 228K+ tokens preserved vs 128K with zero loss
- Agent spawn <500ms (GenServer.start_link), microsecond coordination via PubSub
- 10 cheap agents for ~$0.25 vs ~$4.50 single-Opus
- Self-healing: error classification, automatic diagnosis and repair via ephemeral agents
- Verification loops: autonomous write-test-diagnose-fix cycles with upstream verifiers
- Conversation agents: any agent can spawn brainstorms, design reviews, red team exercises
- Speculative execution: agents work ahead on likely next steps

5 built-in roles: lead, researcher, coder, reviewer, tester. 58 built-in tools. 16 LLM providers, 665+ models. LiveView web UI with 39 components, zero handwritten JavaScript.

328 source files, ~77K LOC, 2,700+ tests.