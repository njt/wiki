---
title: "tolaria"
url: https://github.com/refactoringhq/tolaria
date_fetched: 2026-05-14
section: "Random"
topics:
  - misc
---

# Tolaria: Desktop Markdown Knowledge Base Manager

## Primary Purpose
Desktop application for macOS, Windows, and Linux designed to manage markdown-based knowledge bases. Used for personal knowledge management, organizing company documentation for AI context, and storing assistant memory systems.

## Core Design Principles

1. **Files-first architecture** — Notes exist as plain markdown files, ensuring portability and user data ownership without vendor lock-in.
2. **Git-first approach** — Every vault functions as a git repository, providing version history and remote flexibility without server dependency.
3. **Offline-first operation** — No accounts, subscriptions, or cloud requirements. Works completely offline with zero lock-in.
4. **Open source commitment** — Freely available under AGPL-3.0 license, built by creator Luca Ronin for personal use.
5. **Standards-based format** — Markdown files with YAML frontmatter enable compatibility with external tools.
6. **Types as navigation aids** — Classifications assist discovery without enforcing schemas or validation requirements.
7. **AI-agent compatible** — Supports Claude, Codex CLI, and Gemini integrations while remaining agnostic to AI choice.
8. **Keyboard-centric design** — Optimized for power users prioritizing keyboard efficiency in editor and command palette.

## Technical Stack

- **Frontend**: React with TypeScript
- **Desktop Framework**: Tauri 2
- **Backend**: Rust
- **Languages**: 69.1% TypeScript, 15% JavaScript, 13.5% Rust
- **Package Manager**: pnpm

## Key Statistics
- 10.6k GitHub stars
- 757 forks
- 2,547 commits on main branch
- Creator maintains 10,000+ personal notes

## Installation
Homebrew: `brew install --cask tolaria` or download from release pages.
