# VTcode

An open-source coding agent built in Rust with serious attention to security: OS-native sandboxing (macOS Seatbelt, Linux Landlock + seccomp), tree-sitter-based command validation, comprehensive audit trails. Supports every major LLM provider with automatic failover. The modular Rust architecture and protocol support (ACP, A2A, MCP, ATIF) suggest ambitions beyond just another Claude Code clone. Mac and Linux only.

---

## Key Quotes

(No standout quotes from the README -- it's structured as feature documentation.)

## Key Themes

#coding-agents #security #Rust #open-source #sandboxing

The security-first design aligns with the analysis in [[A Deep Dive on Agent Sandboxes]]. VTcode uses the same OS-level primitives (Seatbelt, Landlock, seccomp) that Codex does, but adds tree-sitter-based command validation on top -- parsing bash commands syntactically before deciding whether to allow execution. This is a deeper defense than pure OS-level sandboxing.

The protocol support is ambitious: ACP (Zed integration), A2A (agent-to-agent), MCP (tool integration), ATIF (session export), Anthropic Messages API compatibility, and Open Responses conformance. This is a bet that coding agents need to interoperate, not just function in isolation.

The Rust architecture (modular monorepo with separate crates for core, LLM, bash runner, indexer, TUI, auth, tools) is the right technical choice for a performance-sensitive tool with security requirements. Rust's memory safety guarantees matter when you're running untrusted commands.

## Critical Analysis

At 589 stars, this is early-stage compared to the tools it competes with. The feature surface is enormous for a young project: every LLM provider, every protocol, multiple sandboxing layers, OAuth, lifecycle hooks, skills system. The risk is spreading too thin.

The Mac + Linux limitation is pragmatic (Windows sandboxing is a different beast) but limits the addressable market. The Ghostty VT PTY snapshots are a nice touch for terminal-native developers but niche.

The tree-sitter-bash command validation is the most technically interesting feature. Most agent sandboxes operate at the OS level (allow/deny filesystem access, allow/deny network). Parsing the command itself before execution gives you a semantic understanding of what the agent is trying to do, not just where it's trying to do it. This could catch malicious commands that are technically within filesystem permissions but operationally dangerous.

Compare with [[Gambit]] for agent evaluation and [[A Deep Dive on Agent Sandboxes]] for the sandboxing landscape. VTcode is building the execution layer; those tools address evaluation and analysis.

---
*Sources: [[summary/vtcode]]*
*Last updated: 2026-05-14*
