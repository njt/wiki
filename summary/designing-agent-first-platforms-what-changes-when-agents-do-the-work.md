---
url: https://azure.microsoft.com/en-us/blog/designing-agent-first-platforms-what-changes-when-agents-do-the-work/
title: "Designing agent-first platforms: What changes when agents do the work"
author: Microsoft Azure Blog
date_fetched: 2026-09-29
date_published: unknown
topics:
  - security-and-sandboxing
  - agent-architecture
---

Microsoft's product essay argues the platform shift from apps that respond to agents that act demands a new architecture: separate the **control plane that governs the agent** from the **execution environment where its work runs**. Agents need first-class identity (Entra Agent ID), tracing and evaluation (Microsoft Foundry), and — critically — a dedicated runtime rather than inheriting the host application's infrastructure.

The core product pitch is **Azure Container Apps Sandboxes**: per-execution hardware-isolated microVMs, spun up in seconds and destroyed after, running as a controllable identity with scoped egress and no stored credentials, with pause/resume that preserves working context across multi-hour tasks. Microsoft claims over a million sandboxes per day internally (GitHub Copilot, Copilot Studio, Security Copilot).

Three case studies anchor the pattern: KPMG's DG Cowork (30,000+ concurrent sandboxes for engagement-isolated tax work), Cognite's Atlas AI (per-user sandboxed code execution against live industrial data), and South Australia's Department for Education EdChat (per-student environments that retired ~50,000 lines of custom code-interpreter state management). The recurring diagnosis: teams stall on the trade-off of broad access (useful but dangerous) versus lockdown (safe but useless), and the resolution is isolation built into the runtime rather than wrapped around it.
