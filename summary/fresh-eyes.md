---
title: "Fresh Eyes"
url: https://github.com/danshapiro/fresheyes
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - guardrails-and-feedback-loops
  - personal-agents
---

# Fresh Eyes

Claude Code plugin providing independent code review using a different AI model than the one generating the code.

## Problem
Using the same model to review its own code has inherent blind spots. Fresh Eyes sends code to "a completely independent model with zero context from your conversation."

## Usage

**Manual Mode:**
- `Review this with fresh eyes` (staged changes)
- `Review commit abc1234 with fresh eyes` (specific commits)
- `Review the files in src/auth/ with fresh eyes` (targeted files)
- Provider selection: `using claude` or `using gpt`

**Automatic Mode:** Pre-commit hook that blocks commits on blocking issues.

## Configuration
Environment variables: `FRESHEYES_PROVIDER`, `FRESHEYES_MODEL`, `FRESHEYES_MODE`.

Prerequisites: Codex CLI (GPT) or Claude Code CLI. Requires committed code before review.

12 stars, MIT License, 100% Shell.
