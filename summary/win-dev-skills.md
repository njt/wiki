---
url: https://github.com/microsoft/win-dev-skills
title: "WinUI agents and skills for Windows app development"
author: Microsoft
date_fetched: 2026-09-22
date_published: 2026-05-13
topics:
  - claude-code
  - coding-agents-and-frameworks
---

Microsoft's win-dev-skills (v0.6.1, preview; earliest tagged release 0.3.0 on 2026-05-13) is an [Agent Plugins 1.0](https://agent-plugins.org/specification) package that installs eight skills and one `winui-dev` orchestrator agent for WinUI 3 / Windows App SDK desktop development into GitHub Copilot CLI, Claude Code, OpenAI Codex, OpenClaw, and OpenCode. It covers the full inner loop — scaffold, design, build, run, test, package, ship — from `winapp new` through a signed MSIX, with Visual Studio explicitly not required.

Its thesis is that skills are *prompts plus playbooks*, and their leverage comes from ground-truth tooling. Two compiled tools ship in-repo: `winmd.exe`, a Native-AOT WinRT/.NET metadata indexer (~8 MB single file) that lets the agent verify an API exists with the intended signature *before* writing code that won't compile, and `Microsoft.WindowsAppSDK.Analyzers.dll`, a Roslyn analyzer that turns the mistakes agents reliably make (UWP namespace leaks, `x:Bind` defaulting to OneTime, field-backed `[ObservableProperty]`, `Window.Current`) into build-time warnings. A thin `BuildAndRun.ps1` injects the analyzer through a reserved MSBuild property and enables crash triage by default.

The repo is deliberately explicit about its own layering: skills are "Tier 3 prose" and the last resort for behavior; tools are "Tier 1 enforcement" and the preferred home — a doctrine encoded in the repo's own multi-subagent PR-review skill. Packaging follows the vendor-neutral Agent Plugins 1.0 spec with per-client compatibility shells (`.claude-plugin/`, `.codex-plugin/`, `openclaw.plugin.json`, a `com.github.copilot/` namespace for the agent), CI byte-verifies the committed analyzer payload against its source, and all binaries are unsigned in preview — an acknowledged trade-off pending NuGet publication. No SemVer until 1.0; marketplace installs track `main`.
