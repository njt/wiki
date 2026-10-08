---
url: https://devblogs.microsoft.com/performance-diagnostics/introducing-windbg-mcp-debug-with-natural-language-grounded-in-evidence/
title: "Introducing WinDbg MCP: Debug with Natural Language, Grounded in Evidence"
author: Microsoft Performance Diagnostics team (Shanthi et al.)
date_fetched: 2026-10-08
date_published: 2026-10-08
topics:
  - mcp-and-tool-protocols
  - software-engineering-craft
---

Microsoft's WinDbg team shipped WinDbg MCP, an MCP server inside WinDbg (version 1.2610.1001.0+) that connects AI clients — VS Code and GitHub Copilot CLI are the supported ones — to a live debugger session over a local proxy. You ask in natural language to root-cause a crash dump, hang, or kernel failure; the client invokes WinDbg tools, and the actions stay visible in the WinDbg UI.

The interesting part is the Diagnostician skill, a five-phase root-cause workflow shipped as part of the windbg plugin in Microsoft's public win-dev-skills catalog. It deliberately treats pattern matches as hypotheses, tests alternatives, and reports candidate root causes with explicit "evidence gaps" — the prompt template literally asks the model to state unverified assumptions.

Security is designed in rather than bolted on: Secure Mode (on by default) blocks loading/executing untrusted code, launching processes, and unsafe file operations; a cross-prompt injection protection uses MCP sampling to classify debugger output for injected instructions. Notably, Claude Code only works if you disable that injection protection, which Microsoft recommends against — because debugger output is untrusted input, and clients that can't do MCP sampling can't run the classifier.
