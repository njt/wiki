---
url: https://pi.dev/docs/latest
title: Pi Coding Agent — Documentation
author: Earendil Inc.
date_fetched: 2026-05-15
date_published: unknown
topics:
  - agent-architecture
---

# Pi Coding Agent — Official Documentation

Pi is a terminal-based coding tool created by Earendil Inc., hosted at pi.dev, and licensed under MIT. It positions itself as "a minimal terminal coding harness" that "stays small at the core while being extended through TypeScript extensions, skills, prompt templates, themes, and pi packages." Available as npm package `@earendil-works/pi-coding-agent`, with source on GitHub at `earendil-works/pi`.

## Quick Start

Two installation methods:
- `curl -fsSL https://pi.dev/install.sh | sh`
- `npm install -g @earendil-works/pi-coding-agent`

After installation, run `pi` in a project directory. Authentication via `/login` for subscription-based providers, or environment variables like `ANTHROPIC_API_KEY`.

## Documentation Structure

### Start Here
- Quickstart: Install, authenticate, first session
- Using Pi: Interactive mode, slash commands, context files, CLI reference
- Providers: Subscription and API-key setup for built-in providers
- Settings: Global and project-level configuration
- Keybindings: Default shortcuts and custom keybinding support
- Sessions: Session management, branching, and tree navigation
- Compaction: Context compaction and branch summarization

### Customization
- Extensions: TypeScript modules enabling tools, commands, events, and custom UI
- Skills: Reusable on-demand capabilities for the agent
- Prompt Templates: Reusable prompts that expand from slash commands
- Themes: Built-in and custom terminal themes
- Pi Packages: Bundling and sharing extensions, skills, prompts, and themes
- Custom Models: Adding model entries for supported provider APIs
- Custom Providers: Implementing custom APIs and OAuth flows

### Programmatic Usage
- SDK: Embedding Pi in Node.js applications
- RPC Mode: Integration over stdin/stdout using JSONL
- JSON Event Stream Mode: Print mode producing structured events
- TUI Components: Building custom terminal UI for extensions

### Reference
- Session Format: JSONL session file format, entry types, and the SessionManager API

### Platform Setup
- Windows, Termux on Android, tmux, Terminal Setup, Shell Aliases

### Development
- Local setup, project structure, and debugging

## Community
- GitHub: github.com/earendil-works/pi/tree/main/packages/coding-agent
- npm: @earendil-works/pi-coding-agent
- Discord: discord.com/invite/3cU7Bz4UPx
- Press Kit: pi.dev/press-kit
- Company: Earendil Inc. at earendil.com
