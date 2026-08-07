---
url: https://devblogs.microsoft.com/dotnet/polyglot-unit-testing-agent/
title: Polyglot Unit Testing Agent
author: Microsoft .NET Team
date_published: 2026-07
date_fetched: 2026-08-07
---

Microsoft's open-source `code-testing-generator` plugin for GitHub Copilot and Claude Code generates unit tests across 12+ languages. It doesn't just write tests — it learns from the repository first (language, framework, conventions, build commands), plans the work at the right granularity (direct / single-pass / iterative), writes tests that follow local conventions, runs them as it works, and verifies the result through lightweight mutation testing, assertion quality checks, and full-suite integration.

The agent is a workflow wrapper around the same underlying model. In benchmarks across 152 tasks from real repositories, it completed 92.1% of tasks vs. 78.9% for stock Copilot — a 63% reduction in failures. The gain came almost entirely from *vague prompts* like "generate unit tests": 88.8% vs. 66.3%, with detailed prompts tied at 96.8%. The agent also passed all 15 tasks asking for tests for a specific code change; stock Copilot passed none.

The workflow helped every model tested (Opus 4.8, GPT-5.5, Haiku 4.5), with Opus gaining the most (+8 wins, 0 losses). Specialized GPT-5.5 reached 90.1% across all tasks, within two points of specialized Opus and 11+ points above stock Opus — suggesting a strong workflow can lift a mid-tier model near the best result.

The agent covers .NET, Python, TypeScript, JavaScript, Java, Go, Ruby, Rust, Swift, Kotlin, PowerShell, and C++. In Python it more than doubled the completion rate (86.7% vs. 40.0%); in Go it passed every task. It's available via the `dotnet/skills` marketplace and open source on GitHub.
