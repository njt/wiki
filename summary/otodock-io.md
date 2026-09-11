---
url: https://otodock.io/
title: "OtoDock — the agentic company OS"
author: OtoDock
date_fetched: 2026-09-11
date_published: unknown
---

OtoDock is a self-hosted "agentic company OS": a multi-user, multi-agent platform that runs Claude Code or Codex as the engine underneath, working on *your* existing Anthropic or OpenAI subscription rather than reselling tokens. The pitch is a company brain — not a coding assistant, but departments of digital employees that persist memory, share workspaces, and act while no one is watching.

Each agent is built from six parts — persona, memory, workspace, knowledge, skills, and tools — and can be shared one of four ways (personal only, personal + shared, shared + personal, shared only). A two-level role system governs both the platform (Admin / Creator / Member) and each agent (Manager / Editor / Viewer). Agents organize into departments, with delegation rules and even agent meetings. Automation is first-class: recurring tasks, webhook triggers, and one-time scheduled jobs write into your workspace and notify you when something needs attention. Agents can hold a phone number (Twilio, or Asterisk/FreePBX on your own server) and run on remote machines — the server sandboxes agents by default, while laptops, workstations, and PCs get full access.

The security posture is the differentiator: "locked down by default, granted on purpose." Every server-side agent runs in a kernel sandbox with its own mount and process namespaces, always-on network isolation (private ranges, LAN, and cloud metadata endpoints unreachable by design), and credentials encrypted at rest and injected per-session so agents can use them but never see them. SSO, two-factor auth, and per-user cost budgets ship from first install.

It is Fair Source: the full source is public, self-hosting is free up to five users, and each release converts to Apache 2.0 after two years. Installation is a one-line script that checks Docker, writes `.env`, and starts the stack (server, dashboard, PostgreSQL, live document preview) on `localhost:8400`. An unusual flourish: the two-minute demo video was "directed, captured and edited by an OtoDock agent."
