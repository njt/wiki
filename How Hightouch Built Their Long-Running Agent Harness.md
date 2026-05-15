# How Hightouch Built Their Long-Running Agent Harness

The best piece I've read on the unglamorous reality of building production agent systems. Hightouch's marketing automation agents need to be "a data scientist, a marketer, and a creative partner all in one" -- and the engineering that makes that work is entirely about context management, not model architecture. The key patterns: separated planning and execution, file buffering for large results, dynamic subagents for messy exploratory work, and fanout to small models instead of RAG.

---

## Key Quotes

> "The ugly truth is that creating a truly capable agent system today has less to do with scaling RL or exotic model architectures and more to do with the deeply unfussy work of actual context engineering."

> "Most of them were too rigid, focusing more on the developer experience of chaining calls together than on solving the core problem of long-form, autonomous reasoning."

> Dynamic subagents are "like using scratch paper on a math test."

## Key Themes

#agents #context-engineering #architecture #production-systems #long-running-agents

The separated planning and execution pattern -- with `make_plan`, `execute_step_in_plan`, and `update_plan` tool calls -- is the same hierarchical approach that [[Scaling Long-Running Agents]] found essential at Cursor. Both teams independently discovered that flat agent coordination fails and you need explicit separation of concerns.

The subagent delegation pattern is particularly sharp: spawn isolated LLM threads for messy work, return only summaries. This is context compression through architecture rather than through summarization algorithms. It connects to the [[Context Rot]] insight that bigger context windows don't help if you're filling them with noise.

The fanout pattern (hundreds of parallel Haiku calls instead of RAG) is counterintuitive and brilliant. Vector similarity search optimizes for the wrong thing. Brute-force classification with small, cheap models is more reliable and often cheaper than maintaining an embedding pipeline.

## Critical Analysis

This is refreshingly honest about what agent engineering actually looks like in production. The DAG critique is spot-on: most agent frameworks are optimized for demos, not for the sparse, non-deterministic tasks that real users need.

The "agents give up" problem they describe is real and under-discussed. Models satisfice -- they find a good-enough answer and stop. The planning infrastructure is essentially a forcing function to keep the agent working past the point where it would naturally stop.

What's missing: monitoring and evaluation. Hightouch describes how they build agents but not how they know whether those agents are performing well in production. The eval framework from [[Demystifying Evals for AI Agents]] would be the natural complement.

The piece is from Amplify Partners (Hightouch's investor), so there's an inherent promotional angle, but the technical detail is genuine and the patterns are validated by independent teams reaching the same conclusions.

---
*Sources: [[raw/how-hightouch-built-their-long-running-agent-harness]]*
*Last updated: 2026-05-14*
