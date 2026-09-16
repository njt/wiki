---
url: https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/stop-letting-agents-run-the-workflow/4550068
title: "Stop Letting Agents Run the Workflow"
date_fetched: 2026-09-16
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

A Microsoft Foundry blog post arguing that the most common way enterprise multi-agent systems fail is not hallucination but ownership drift: no single deterministic component owns the business process, so agents quietly become the process owners. The opening war story is a five-agent access-request system that grants a contractor 30 days of standing admin access instead of 8 hours — every agent doing roughly what it was asked, nothing hallucinated, and the system still wrong.

The proposed pattern is "Deterministic Spine, Agentic Leaves": the business process is a state machine owned by deterministic workflow code (states, transition guards, approval gates, idempotency, retries, audit), while agents are bounded workers inside the states, doing classification, extraction, drafting, and summarisation. Agents may recommend but never move the workflow, draft but never approve, prepare payloads but never execute them. The post grounds the pattern in converging vendor guidance (Microsoft Agent Framework's workflows-vs-agents split, Anthropic's, LangGraph's, OpenAI's), then gives a four-layer Foundry reference architecture: workflow spine, schema-contracted agent leaves, deterministic policy gates separate from AI safety guardrails, and a controlled action plane of strict, idempotent, schema-validating tools.

It closes with five design mistakes, a six-step checklist (map the process before naming an agent, explicit state objects, per-leaf evals, approval before execution, verify against the system of record), and the slogan: workflow owns the process, policy owns the permission, tool owns the action, agent owns only its assigned reasoning task.
