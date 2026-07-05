# PiClaw

Packages the Pi Coding Agent into a self-hosted Debian sandbox with a streaming web UI, persistent sessions, scheduled tasks, workspace tooling, and optional WhatsApp support. A single Docker container that gives you chat, code editor, terminal, file viewers, and automation in one web app. Built on Bun, backed by SQLite.

---

## Key Themes

#self-hosted #agent-infrastructure #docker #web-ui #pi-agent

PiClaw is notable for taking the "self-hosted agent" idea seriously -- not just "run a model locally" but the full operational package: authentication (passkeys, TOTP), persistent storage, scheduled tasks, MCP support, and deployment docs for Azure VMs. It's thinking about hosting as a first-class concern, which most agent tools ignore.

This connects to the [[maclocal-api]] (local inference) and [[What I learned building an opinionated and minimal coding agent]] (the pi agent it packages) threads. Zechner built the minimal agent; PiClaw (by Rui Carmo) wraps it for deployment. The relationship between the two illustrates the "build vs. deploy" gap in agent tooling -- building a good agent and operating one reliably are different problems.

## Critical Analysis

Strong: thinking about self-hosting as a first-class concern is rare and valuable. The authentication and access control features (passkeys, TOTP, session-scoped SSH) show awareness that exposing an agent via web UI is a security decision, not just a convenience feature. The "never delete messages.db" warning is honest about data durability.

Weak: TypeScript on Bun is a less proven runtime choice for long-lived server processes than Go or Rust alternatives in this space (compare [[Zeroclaw]], [[serf]]). The feature list (chat + editor + terminal + file viewers + browser automation + image processing + scheduled tasks + WhatsApp) feels like scope creep for what started as a coding agent wrapper.

The WhatsApp integration is interesting as a deployment channel -- messaging apps are where non-developers actually live. See also [[MimiClaw]] which uses Telegram for the same reason.

---
*Sources: [[summary/piclaw]]*
*Last updated: 2026-05-14*
