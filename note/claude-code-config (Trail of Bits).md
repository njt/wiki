# claude-code-config (Trail of Bits)

Trail of Bits' opinionated defaults for Claude Code, covering sandboxing, permissions, hooks, skills, MCP servers, and usage patterns developed across security audits, development, and research. This is what serious, security-conscious Claude Code usage looks like in practice.

---

## Key Quotes

> "A session that stays within its context budget produces better code than one that compacts three times to limp across the finish line."

> "Guardrails, not walls."

## Key Themes

#claude-code #configuration #security #hooks #sandboxing #best-practices

The hooks system is the most valuable contribution. Structured execution triggers at decision points -- PreToolUse blocks dangerous commands before execution, PostToolUse audits after. Exit codes determine blocking behavior. This is the mechanical enforcement that [[CLAUDE.md (Universal)]] cannot provide through prompt instructions alone.

The security posture is thorough: filesystem and network isolation in `/sandbox`, deny rules blocking SSH keys, cloud credentials, package tokens, git credentials, and crypto wallets. Devcontainer option for complete isolation, remote droplets via dropkit for full machine isolation.

The MCP server selection is curated and practical: Context7 for library docs (no API key), Exa for web/code search, plus specialized servers for smart contracts (slither-mcp) and reverse engineering (pyghidra-mcp).

Usage patterns reveal hard-won wisdom: weekly `/insights` reviews, `/clear` over `/compact`, aggressive checkpointing, autonomous `/fix-issue` and `/review-pr` commands. The fast mode guidance (2.5x faster, 6x cost, interactive debugging only) is the kind of calibration you only get from heavy usage.

Connects to [[Pre-Commit Lint Checks]] (both are about mechanical enforcement), [[claude-ctrl]] (which takes the enforcement idea further with SQLite-backed policy), and [[CLAUDE.md (Universal)]] (which this supersedes for anyone serious about configuration).

## Critical Analysis

This is the reference implementation for Claude Code configuration. If you're using Claude Code professionally and haven't at least reviewed this repo, you're leaving quality and security on the table. The main limitation is that it's deeply Trail of Bits-flavored -- security audit workflows, Rust/Python/Node toolchains, their specific MCP servers. You'll need to adapt rather than adopt wholesale. But the hooks system, the sandboxing approach, and the permission deny rules are universally applicable.

---
*Sources: [[summary/claude-code-config]]*
*Last updated: 2026-05-14*
