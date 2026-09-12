---
title: "session-analysis"
url: https://github.com/lhl/fsr4-rdna3-optimization/tree/main/session-analysis
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Session Analysis

Leonard Lin's tools for analyzing AI coding assistant session data from Claude Code and Codex CLI, developed during FSR4 kernel optimization work.

Key insight: "When using AI coding assistants for non-trivial work (like our FSR4 kernel optimization campaign), it's useful to understand how long things actually took."

Important cost nuance: "Claude Code cache tokens dominate total API token counts...a session showing 4M total API tokens may have only 6K regular input + 13K output."

Sessions stored as JSONL files in standard locations (~/.claude/projects/ and ~/.codex/sessions/). Both tools use distinct entry types. Script enables queries about session duration, token usage, and collaborative phases through JSON piping and filtering. Auto-discovers sessions and is extensible.

Tracks wall time vs active compute time, token consumption patterns, and how human-AI collaboration unfolded.
