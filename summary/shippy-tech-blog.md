---
url: https://huggingface.co/blog/allenai/shippy-tech-blog
title: "What building Shippy taught us about building agents"
author: Kyle Wiggers (Ai2Comms)
date_fetched: 2026-07-18
date_published: 2026-07-15
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

Shippy is a maritime AI agent built by the Skylight team at Ai2 (Allen Institute for AI). It provides real-time ocean domain awareness to government agencies and NGOs, drawing on Skylight's vessel tracking, satellite imagery, and boundary data. The article is a technical postmortem on what made the agent reliable enough for operational use.

The team decomposes the agent into three parts. The **soul** is the system prompt — persona and behavioral guardrails, including explicit boundaries like refusing to make legal determinations. **Skills** are plain Markdown files (following the agent-skills spec used by Claude Code) describing how to handle specific request types: querying vessel data, looking up EEZ/MPA boundaries, interpreting track data, and generating map links. A single query can invoke multiple skills in one turn. **Config** covers runtime settings (the OpenClaw harness, Claude Opus 4.6, API keys), separated so changes don't require a rebuild.

The central insight is that agents are nondeterministic, but their tools can be made reliable. Rather than letting Shippy construct raw API calls — which produced a steady stream of subtle bugs — the team built a purpose-built CLI (`skylight events search`) with typed flags, extensive `--help` text, and automatic handling of auth and pagination. Output always goes to a local JSON file, never through shell pipes. Each layer (API test suite → CLI → skill docs) narrows what the next layer can get wrong.

Shippy runs on **Mothership**, an in-house hosting platform that provisions a dedicated Kubernetes pod per user session. The user's JWT is injected at provision time, scoping all data access. Files written during analysis are session-scoped and never shared; network access is restricted to needed services only. This handles the data-isolation requirements of serving hundreds of agencies across 70+ countries.

For evaluation, the team rejected static benchmarks in favor of whole-agent eval against live data. Subject-matter experts write scenarios and rubrics; a natural-language prompt runs through the sandbox; an LLM judge grades each criterion 0–1 with written reasoning; experts annotate responses for ground truth. The process runs through **Harbor**, an open evaluation framework, producing timestamped results and delta reports per build. Recent findings included Shippy overstepping into tactical recommendations on patrol-planning tasks and one case of hallucinating a nonexistent CLI command.

Planned improvements include agent-driven map UI control, model routing (small models for simple lookups, frontier models for complex investigations), and cross-thread memory. The Mothership architecture is already being reused by other Ai2 platforms (EarthRanger, OlmoEarth).
