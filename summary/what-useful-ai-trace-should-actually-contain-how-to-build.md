---
url: https://www.telerik.com/blogs/what-useful-ai-trace-should-actually-contain-how-to-build
title: "What a Useful AI Trace Should Actually Contain (and How to Build One)"
author: Nikolay Iliev
date_fetched: 2026-09-20
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

A vendor implementation guide from Progress/Telerik arguing that standard OpenTelemetry tracing is structurally insufficient for debugging AI agents: a span that records a 1.3-second successful LLM call tells you the infrastructure worked but nothing about what the model received, returned, called, cost, or whether it was right. The article names the industry problem "trace noise" — auto-instrumentation captures everything, so LLM spans drown among HTTP, auth, and database telemetry — and contrasts trace volume (easy) with trace hygiene (deliberate design).

The bulk of the post is a Python SDK walkthrough for the Progress AI Observability Platform built on six trace elements: prompt sent, completion returned, tool invocations with inputs and outputs, token counts, estimated cost, and a quality signal. Four implementation steps: initialize instrumentation before importing LLM libraries (patching happens at instrument time), set `trace_content=True` for debugging environments, apply a four-decorator structure (`@agent`, `@workflow`, `@tool`, `@task`) so traces mirror agent logic rather than flat span lists, and flush telemetry on shutdown — especially in serverless, where buffered spans are routinely lost.

Two architecture choices stand out: cost calculation lives in the collector rather than application code (so provider price changes are config, not deploys), and per-request tags via `propagate_attributes` enable tenant- and release-level filtering. A tradeoff table covers the PII/debuggability tension of capturing prompt content in production.
