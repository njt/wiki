---
url: https://github.com/boldsoftware/shelley
title: "Shelley"
author: Bold Software, Inc.
date_fetched: 2026-09-08
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

# Shelley: a coding agent for exe.dev

Shelley is a mobile-friendly, web-based, multi-conversation, multi-modal, multi-model, single-user coding agent built for — but not exclusive to — [exe.dev](https://exe.dev/). It ships without authorization or sandboxing: "bring your own." The rationale is explicit in the README: mobile-friendly because ideas come any time, web-based because terminal scrollback "is punishment for shoplifting in some countries," multi-modal because screenshots and charts are necessary, multi-model to benefit from all the innovation going on, and single-user "because it makes sense to bring the agent to the compute."

The stack is Go for the backend, SQLite for storage, and TypeScript with Vue 3 + PrimeVue for the UI, compiled into a single binary that embeds the frontend. The core data model is simple: Conversations have Messages, which may come from the user, the model, the tools, or the harness — all persisted to the database and pushed to the UI over a Server-Sent Events (SSE) endpoint. Shelley is partially based on the authors' previous coding agent, [Sketch](https://github.com/boldsoftware/sketch), and is itself substantially written by Shelley, Sketch, Claude Code, and Codex.

The name is a shell pun ("the main tool it uses is the shell, and I like putting '-ey' at the end of words") with an ironic nod to Percy Bysshe Shelley's "Ozymandias." Shelley is Apache-licensed and requires a CLA for contributions; new releases are cut automatically on every commit to `main` under the version scheme `v0.N.9OCTAL`.
