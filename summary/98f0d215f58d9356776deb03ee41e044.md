---
url: https://gist.github.com/njt/98f0d215f58d9356776deb03ee41e044
title: "AI Agents In-Depth — Function Calling, MCP and Tool Use Under the Hood"
author: Alan Smith
site: NDC Conferences
date_fetched: 2026-08-07
topics:
  - mcp-and-tool-protocols
  - agent-architecture
---

Alan Smith delivers a 60-minute deep-dive into how LLM tool calling actually works at the protocol level, plus a pragmatic tour of MCP, multi-agent systems, skills, and the sharp edges of agentic development. The talk is demo-heavy — Postman-level JSON, a pizza-ordering agent, a vibe-coded website builder, a RAG pipeline with Wikipedia, and a DJ MCP server — each chosen to surface a specific failure mode or design insight.

The foundational distinction: **the LLM doesn't call tools**. It selects the tool and the parameters; the agentic application makes the actual call. Smith walks through the raw OpenAI function-calling JSON to make this concrete, then builds up through SDKs, frameworks (LangChain, Semantic Kernel, Agent Framework), and MCP.

On MCP, he delivers a clarifying correction: the Model Context Protocol is *not* a universal format for the JSON you send to models. That's the "model context," and every provider handles it differently via their SDKs. MCP is a protocol for distributed tool servers — tools, resources, templates, and notifications — and the streaming HTTP transport exists specifically because of those notifications. His advice: MCP is cool, but you don't need it if simple function calling suffices. It's an architectural decision, like function call vs. microservice.

Several failure modes are demonstrated live: the **hammer-nail problem** (LLMs using DALL-E to generate QR codes because "it's an image generator"), **tool trust** (a calculator returning wrong answers that the LLM accepts), **non-determinism** (same prompt, different pizza orders every time), and **security leakage** (a credit card number sent to a tool without hesitation — though behaviour has since changed on some providers). Smith notes that non-determinism makes testing hard and that provider-side behaviour changes mean your system will do something different next week than it does today.

Other threads: multi-agent setups keep conversation histories smaller (each agent maintains its own context); skills in Copilot are efficient context management that loads on demand; vibe coding can build real apps but has pitfalls; and RAG with tool calling is strictly better than naive pre-tool-calling RAG because the LLM can decide *when* and *what* to search.
