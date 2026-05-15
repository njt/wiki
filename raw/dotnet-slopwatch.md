---
title: "dotnet Slopwatch"
url: https://github.com/Aaronontheweb/dotnet-slopwatch
date_fetched: 2026-05-14
section: "C# and .NET"
---

# Slopwatch: LLM Anti-Cheat for .NET

## Purpose
A .NET tool designed to detect "reward hacking" behaviors where AI coding assistants take shortcuts instead of properly solving problems.

## Detection Capabilities
- Disabled tests (Skip, Ignore, #if false)
- Warning suppressions via pragma or SuppressMessage
- Empty catch blocks swallowing exceptions
- Arbitrary delays masking timing issues (Task.Delay, Thread.Sleep)
- Project-level warning suppression patterns
- Central Package Management bypasses

## Integration
- Runs as Claude Code hook during AI-assisted coding
- Integrates with CI/CD pipelines (GitHub Actions, Azure DevOps)
- Baseline tracking to catch only new issues
- JSON output for programmatic consumption
- Configurable suppression rules via .slopwatch/config.json

## Workflow
1. Initialize baseline from existing code (`slopwatch init`)
2. Commit baseline to repository
3. Run analysis on new changes (`slopwatch analyze`)
4. Update baseline when intentional code is added (`slopwatch analyze --update-baseline`)

## Key Quote
"When LLMs generate code, they sometimes take shortcuts that make tests pass or builds succeed without actually solving the underlying problem."

Apache 2.0 licensed.
