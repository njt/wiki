---
title: "Gambit"
url: https://github.com/bolt-foundry/gambit
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - guardrails-and-feedback-loops
---

# Gambit: Agent Harness Framework

## Purpose
Gambit is a synthetic scenario and evaluation layer for agent systems. It enables developers to create realistic test scenarios, validate their quality, run agents against them, grade behavior, capture trace evidence, and convert failures into regression test suites.

## Key Features

**Core Capabilities:**
- Generate synthetic scenarios for realistic user, tool, workflow, policy, and edge case testing
- Evaluate scenario data for realism, coverage, difficulty, and clarity
- Execute agent evaluations against native Gambit, Mastra, LangGraph, OpenAI, or custom agents
- Grade agent behavior from transcripts, artifacts, traces, and typed outputs
- Diagnose failures using trace and permission evidence
- Regenerate regression suites from failures to prevent behavior regression

**Supported Workflows:**
1. Native Gambit path: Define agents in Gambit with integrated verification
2. Bring-your-own-agent: Use external frameworks (Mastra, LangGraph) with Gambit's evaluation layer
3. Pull request gates: Run behavior checks on every PR with automated grading

## Architecture Overview

Gambit operates as "the test-data engine, grader loop, local reproduction harness, and CI behavior check" for agent systems. It provides:

- **CLI tools** for running agents, creating REPL sessions, and grading outputs
- **Simulator UI** (Debug interface) for streaming runs and trace visualization
- **TypeScript/Deno library** exports for programmatic agent definitions
- **Markdown/TOML deck format** for declarative agent specification

## Quick Start

Basic commands:
- `npx @bolt-foundry/gambit demo` - Download examples and set up environment
- `npx @bolt-foundry/gambit run <deck>` - Execute agent once
- `npx @bolt-foundry/gambit repl <deck>` - Interactive terminal loop
- `npx @bolt-foundry/gambit-simulator serve <deck>` - Launch browser-based Debug UI

## Technical Stack

- **Language:** TypeScript (99.5% of codebase)
- **Runtime:** Node.js 18+ or Deno
- **Requirements:** `OPENROUTER_API_KEY` for model-powered agents
- **Default execution:** Worker sandbox mode (can be disabled with `--no-worker-sandbox`)

## Agent Definition Examples

Gambit supports three definition patterns:
1. **Markdown-based** (model-powered): Declarative TOML front matter with instruction text
2. **TypeScript compute agents**: Direct code execution without LLM calls
3. **Composite agents**: Parent agents calling child actions via typed tools

All definitions use schemas for input/output validation and support structured tracing.
