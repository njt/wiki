---
title: "claude-code-config"
url: https://github.com/trailofbits/claude-code-config
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - claude-code
  - security-and-sandboxing
---

# Trail of Bits Claude Code Config

Opinionated defaults, documentation, and workflows for Claude Code at Trail of Bits, covering sandboxing, permissions, hooks, skills, MCP servers, and usage patterns for security audits, development, and research.

## Core Setup Components
- **Terminal**: Ghostty recommended for native GPU rendering
- **Development Tools**: Homebrew packages (jq, ripgrep, fd, ast-grep, shellcheck, etc.), Python tools via uv, Rust toolchain, Node.js tools, LM Studio for local models
- **Settings Configuration**: Privacy controls, agent teams, extended thinking, permission rules, status line display
- **Global CLAUDE.md**: Development philosophy, code quality standards, language-specific toolchains

## Sandboxing & Security
- Built-in `/sandbox` environment with filesystem and network isolation
- Read/write permission deny rules blocking SSH keys, cloud credentials, package tokens, git credentials, crypto wallets
- Devcontainer option for complete project isolation
- Remote droplet support via dropkit for full machine isolation

## Hooks System
Structured execution triggers at decision points:
- **PreToolUse**: Block commands before execution
- **PostToolUse**: Log and audit actions after execution
- **UserPromptSubmit**: Intercept user input
- **Stop**: Review final responses before completion

"Guardrails, not walls."

## Notable Quotes
"A session that stays within its context budget produces better code than one that compacts three times to limp across the finish line."

## Usage Patterns
- Weekly `/insights` reviews for workflow improvements
- Project-level CLAUDE.md layered on global standards
- `/clear` over `/compact` for context management
- Autonomous `/fix-issue` and `/review-pr` commands
- Fast mode toggle for interactive debugging only (2.5x faster, 6x cost)

## MCP Servers
- Context7: Library documentation lookup
- Exa: Web and code search
- Specialized: slither-mcp (smart contracts), pyghidra-mcp (reverse engineering), Serena (code navigation)

2k stars, Shell (100%), MIT-adjacent. Related: trailofbits/skills, trailofbits/dropkit.
