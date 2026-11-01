---
url: https://github.com/HazAT/ultimate
title: "Ultimate"
author: Daniel Griesser (HazAT)
date_fetched: 2026-11-02
date_published: 2026 (ongoing personal project)
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

Ultimate is Daniel Griesser's personal, MIT-licensed coding agent: Pi's coding agent (the @earendil-works/pi-coding-agent stack, on Pi Durable) served in a local web UI, specialized per project. Everything that is not core plumbing is an extension folder with a `server.ts` (agent facet) and/or `web.tsx` (UI facet); extensions, subagent definitions, and skills live in `<project>/.ultimate/`, are committed with the repo so everyone on the project boots the same specialized agent, and hot-reload while the conversation runs.

The architecture's spine is durability and self-hosting: every conversation, tool call, and subagent is committed to SQLite before it is shown, so the agent can rewrite its own UI, extensions, and even its core server while a session continues. The supervisor (`bin/`) is deliberately frozen — changing it requires a human restart — while a `restart` tool boots and type-checks a new core process next to the old one and rolls back on failure.

Work is organised as a forest of conversation trees (the `lineage` extension, the largest at ~1,700 lines): one root per project holds a read-only "tally" of branch intents, conversation abstracts, and landings, plus slug-based project memories; branches start from a memory brief or the tally; coordinators plan and verify while subagents do bounded execution. Instance-to-instance messaging, codemode reads of other projects, and Tailscale exposure let several Ultimates act as a hub-and-worker fleet. Fifty-plus built-in extensions ship on by default and can be disabled per project or machine in `settings.json`.
