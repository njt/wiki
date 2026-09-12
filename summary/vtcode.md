---
title: "VTcode"
url: https://github.com/vinhnx/VTCode
date_fetched: 2026-05-14
section: "Random"
topics:
  - security-and-sandboxing
---

# VT Code: Open-Source Coding Agent

## Primary Purpose
Command-line AI coding agent with emphasis on security, multi-provider support, and sophisticated code understanding. Mac and Linux only.

## Key Features

**Security & Safety**
- Multi-layered defense: tree-sitter-bash command validation and execution policies
- OS-native sandboxing via macOS Seatbelt and Linux Landlock + seccomp
- Configurable human-in-the-loop approval systems
- Comprehensive audit trails for command execution

**AI Provider Support**
GitHub Copilot, OpenAI, Anthropic, Google Gemini, DeepSeek, OpenRouter, Z.AI, Ollama, and more. Automatic failover.

**Code Intelligence**
- LLM-native semantic analysis across multiple programming languages
- Tree-sitter integration for syntax-aware code understanding
- Smart refactoring and navigation tools

**Integration Capabilities**
- Agent Client Protocol (ACP) for Zed IDE integration
- Agent2Agent (A2A) Protocol for inter-agent communication
- Anthropic Messages API compatibility layer
- Open Responses specification conformance
- Agent Trajectory Interchange Format (ATIF) for session export
- MCP (Model Context Protocol) support

**Advanced Features**
- OAuth 2.0 authentication with PKCE flows
- Comprehensive skills system (open Agent Skills standard)
- Lifecycle hooks for custom shell commands during agent events
- Context management with token budget tracking
- Rich terminal UI (Ratatui)
- Ghostty VT PTY snapshots

## Architecture
Modular Rust monorepo:
- `vtcode-core`: Primary agent logic
- `vtcode-llm`: LLM provider abstraction
- `vtcode-bash-runner`: Secure command execution
- `vtcode-indexer`: Code indexing and search
- `vtcode-tui`: Terminal UI (Ratatui)
- `vtcode-acp`: Agent Client Protocol
- `vtcode-auth`: OAuth and credential management
- `vtcode-tools`: Tool specifications and policies

## Installation
Native installers, Cargo, Homebrew. MIT licensed. 589 stars.
