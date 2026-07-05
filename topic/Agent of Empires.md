# Agent of Empires

A session manager for running multiple AI coding agents in parallel, built in Rust. It wraps tmux sessions around Claude Code, Codex, Gemini CLI, and a dozen other agents, giving you a TUI, web dashboard, and even a mobile-first "cockpit" view. The killer feature is git worktree integration for branch isolation -- each agent works on its own branch without stepping on the others.

---

## Key Quotes

> No standout quotes -- it's a tool README, not an essay.

## Key Themes

#multi-agent #session-management #tooling #git-worktrees #parallel-development

The "superagent" problem -- how to run multiple AI agents productively without them colliding -- is one of the open questions in agentic coding. AoE takes the pragmatic infrastructure approach: tmux for persistence, worktrees for isolation, Docker for sandboxing. It's the plumbing layer that makes [[Claude Code on the Go]]'s "six agents, six features, one phone" vision practical.

The remote access feature (Tailscale Funnel or Cloudflare Tunnel with QR code auth) is clever and connects to the same mobile-development pattern.

## Critical Analysis

Great name, questionable metaphor (as the annotation notes). The real question is whether managing 12 different agent CLIs through a unified session manager is the right abstraction. Most serious users seem to converge on one or two agents and go deep rather than spreading across many. The 2.2k stars suggest real traction, though. The Cockpit feature (swipe-to-approve on mobile) hints at a future where code review is something you do from your phone while walking -- which is either exciting or terrifying depending on your perspective. Compare with [[Awesome Agentic Patterns]] for the theoretical patterns this tool implements in practice.

---
*Sources: [[summary/agent-of-empires]]*
*Last updated: 2026-05-14*
