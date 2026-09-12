---
url: https://github.com/asamassekou10/ship-safe
title: Ship Safe
author: asamassekou10
date_fetched: 2026-08-06
topics:
  - security-and-sandboxing
---

# Ship Safe — Summary

Ship Safe is an open-source (MIT) AI security scanner CLI, distributed as an npm package (`npx ship-safe`), that targets the intersection of traditional application security and AI-agent-specific threats. It ships with 29 parallel scanning agents covering 80+ attack classes across traditional code vulnerabilities, supply chain, CI/CD, secrets, and AI/LLM security domains (prompt injection, agent hijacking, MCP misconfiguration, memory poisoning, RAG poisoning, and more).

The tool runs entirely locally by default — no signup, no API key required for core scanning. AI-backed modes (deep analysis, GPT-Red teaming) use a configured provider when available. Findings are scored on a 0–100 scale mapped to letter grades with confidence-weighted deductions. The scoring engine uses a soft-cap curve (`d = weight × raw / (raw + weight)`) so categories never saturate to zero, keeping scores comparable across runs of any size.

Key architectural decisions: all agents run in parallel (6-way concurrency by default), each with a 30s timeout. A suppression floor prevents inline `ship-safe-ignore` comments from silencing critical findings, on the reasoning that any AI agent that can write source code can also write the suppression comment. An extensible plugin system lets users drop custom agents into `.ship-safe/agents/` for automatic loading. Post-processors include a VerifierAgent (secrets liveness, finding downgrade), a DeepAnalyzer (LLM taint analysis with 3-tier cascade model routing), and a SecurityMemory (false-positive learning across scans).

The project is at v9.7.0 with ~44K lines of JavaScript across 127 source files. It targets AI-native applications specifically — MCP server configs, Claude Code hooks, `.cursorrules`, `CLAUDE.md` files, managed agent configurations, and Hermes agent deployments — making it one of the few scanners that treats AI-coding-agent surface area as a first-class security domain.
