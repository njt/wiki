---
url: https://devblogs.microsoft.com/dotnet/how-uno-platform-uses-dotnet-mcp-ai-to-build-high-quality-apps/
title: "How Uno Platform Uses .NET, MCP and AI to Build High-Quality Apps"
author: Sam Basu
date_fetched: 2026-09-15
topics:
  - mcp-and-tool-protocols
  - guardrails-and-feedback-loops
---

Sam Basu explains how Uno Platform built two MCP servers to let AI agents write and verify cross-platform .NET applications. The central design decision is a split by "lifetime": a stateless, hosted, HTTP docs server (versioned with the documentation, not the SDK) answers what is true about the framework, while a stateful, stdio-launched app server on the developer's machine bridges to the running app to answer what is actually happening right now.

The app server gives agents four capabilities — run the app with Hot Reload, see it via screenshots and XML visual-tree snapshots, act on it through pointer/keyboard tools and automation peers, and check its own connection health. Basu argues the visual tree is the tool that earns its keep: pixels detect that something is wrong, structure diagnoses which element is at fault. This is pitched as the missing Playwright-equivalent for native .NET apps, closing the gap between cheap generation and expensive verification.

The post also distills production lessons from building both servers on the MCP C# SDK: pick transport from topology, budget for tool definitions as a permanent tax on the context window (6.4k tokens for docs, 1.5k for the app server), and treat tool descriptions as prompts that steer model selection — `uno_app_pointer_click` explicitly tells the model to prefer automation peers. Skills layer curated procedure on top of atomic tools, and the whole stack powers Uno Platform Studio 3.0's browser-based app generation with Roslyn compilation and hot reload.
