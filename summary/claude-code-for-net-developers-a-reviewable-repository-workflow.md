---
url: https://www.devleader.ca/2026/10/12/claude-code-for-net-developers-a-reviewable-repository-workflow
title: "Claude Code for .NET Developers: A Reviewable Repository Workflow"
author: DevLeader
date_fetched: 2026-10-14
date_published: 2026-10-12
topics:
  - claude-code
  - agent-coding-workflow
---

DevLeader's October 2026 essay frames Claude Code in a .NET repository around one question: how do you make an agent work inside existing constraints (conventions, authentication, tests, build environment) and return something a developer can actually evaluate? The answer is a "reviewable loop": establish the environment, identify the relevant code, describe the contract, authorize bounded work, and inspect evidence.

The piece is grounded in freshly accessed official Claude Code documentation (checked 2026-10-09) and walks a single worked example — an orders endpoint that must reject a negative page-size — through each stage. Along the way it makes careful product distinctions: native Windows vs WSL 2 (sandboxing only in the latter), VS Code vs Visual Studio, `CLAUDE.md`/`AGENTS.md` as context mechanisms rather than access controls, the `csharp-lsp` plugin vs the separate `csharp-ls` binary, and deny/ask/allow permission rules with deny evaluated first.

Two reframings carry the essay's weight. First, capabilities are not evidence of success: reading a repository doesn't mean every constraint was found, a green test doesn't mean the intended regression was covered — the workflow's job is to force the agent to report those distinctions rather than flattening them into "done." Second, instructions must be separated from authority: prose guidance influences model choices, but runtime permissions, constrained tools, and credential isolation are what actually enforce boundaries. The handoff standard is a chain of evidence — command, revision, environment, result, skipped coverage — where "sample authored" and "sample executed successfully" are different statuses.
