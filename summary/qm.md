---
url: https://github.com/yc-software/qm
title: "QM — a multiplayer agent harness for work"
author: yc-software
date_fetched: 2026-08-14
---

QM is an open-source (MIT) multiplayer agent harness for whole-company use, built for startups. Where most agents are personal assistants, QM gives every employee their own isolated workspace *and* lets them collaborate with the agent in Slack channels, group messages, and projects. Each person and each room owns scoped memory, files, a keychain view, permissions, crons, web apps, and a durable sandbox.

The core is a headless TypeScript service (Fastify on Node) that runs the agent loop and can be driven by any of several harnesses — Pi, OpenCode, Codex, or Claude Code — so a deployment isn't tied to one vendor. A Postgres layer holds sessions, memory, and a work queue. The agent has a small, fixed tool surface; the `execute` tool runs commands in the scope's own isolated sandbox, its durable "computer" where installed tools stay installed. The web UI, admin panel, and portal are optional plugins over the core's HTTP API; Slack is an in-process plugin the core supervises.

Three security postures — strict (every tool call pauses for approval), auto (a classifier screens provenance-labelled external data), and dangerous (no screening) — combine with a predeclared command policy (approval rules and hard denials for recursive deletes or destructive SQL) that applies in every posture. An org picks one posture; narrower scopes can only tighten it.

Org-specific configuration, tools, skills, sandbox image, and infrastructure live in a **deployment directory** that the `qm` CLI validates and deploys, so deployments run in the operator's own cloud account without a source checkout. Alternatively, a **private fork** (a plain clone, never GitHub's fork button) keeps core byte-identical to upstream while holding private customizations under `deploy/layers/<org>/`.
