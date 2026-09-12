---
title: "serf"
url: https://github.com/prime-radiant-inc/serf
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - coding-agents-and-frameworks
---

# Serf: Non-Interactive Coding Agent

From Prime Radiant (Jesse's company). Give it a task, it does the work. Supports OpenAI, Anthropic, and Google models. Go-based.

## Core Architecture

Structured loop: reads files, modifies code, runs commands, searches repositories -- all while maintaining conversation state. Sessions persist to .serf/sessions/ for resumption.

## Key Components

- File reading and writing
- Command execution
- Code search functionality
- Structured output schema support

## Execution Modes

- Single-shot LLM calls via llmcall (no looping, no tool usage)
- Full agent loop with serf (iterative task completion)
- Web orchestration via serf-hub for concurrent sessions
- Terminal UI dashboard via serf-tui

## Session Management

Auto-saves state. Resume interrupted work with --resume-last. Carry forward context with --resume-with. Override model choices during resumption.

## Technical Stack

Go (70%), Python (16.6%), JavaScript (8.8%). Multi-provider support: OpenAI, Anthropic, Google, Minimax, OpenRouter, Kimi, GLM, Ollama.

## Notable Features

Hub architecture with loopback-only daemons and rendezvous files at ~/.serf/run/. Web interface supports edit-to-fork workflows, transparent session resumption, Cmd-K cross-session search.

## Lineage

Descends from Kilroy by Dan Shapiro, originally developed for the StrongDM Attractor project.
