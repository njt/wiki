# Claude Code on the Go

A mobile-first development workflow: Claude Code agents running on a cloud VM, controlled entirely from an iPhone. The setup is a Vultr VM ($0.29/hour, on-demand), Tailscale for networking, Termius + mosh for resilient SSH, tmux for persistence, and Poke for push notifications. The result: "six agents, six features, one phone."

---

## Key Quotes

> "No laptop, no desktop -- just Termius on iOS and a cloud VM."

> "The loop is: kick off a task, pocket the phone, get notified when Claude needs input."

> "Without notifications, you'd constantly check the terminal. With them, you can walk away."

## Key Themes

#mobile-development #cloud-vm #async-workflow #tailscale #notifications #parallel-agents

The push notification loop is the key insight. Without it, mobile development is constant terminal-checking. With it, you can treat agent runs as background jobs that interrupt you only when they need something. This is the same async pattern that makes [[Agent of Empires]]'s multi-agent approach practical.

The cost model ($0.29/hour, start/stop on demand via iOS Shortcuts) makes this accessible rather than expensive. Combined with git worktrees for branch isolation, you get parallel development from a phone.

## Critical Analysis

This is a beautiful hack, and the author clearly uses it. The question is whether "development from your phone" is solving a real problem or creating an impressive-looking workflow for a need that rarely arises. How often do you actually *need* to write software from your phone? The honest answer is probably "almost never" -- but the interesting reframe is that when agent runs take hours (as in [[Building low-level software with only coding agents]]), the phone becomes a monitoring interface, not a coding interface. You're not *developing* from your phone; you're *supervising* from your phone. That's a meaningfully different and more defensible use case.

The Tailscale + mosh + tmux stack is solid and well-chosen. Each component solves a specific problem (networking, connection resilience, session persistence) without overlap.

---
*Sources: [[summary/claude-code-on-the-go]]*
*Last updated: 2026-05-14*
