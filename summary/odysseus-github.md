---
url: https://github.com/odysseus-dev/odysseus
title: "Odysseus — self-hosted AI workspace (odysseus-dev/odysseus)"
author: odysseus-dev
date_fetched: 2026-10-05
date_published: 2026-10-05
topics:
  - personal-agents
  - coding-agents-and-frameworks
---

Odysseus is a self-hosted AI workspace covering chat, agents, deep research, documents, email, notes, calendar, and local-model workflows in one deployable unit. It ships as Docker (`ghcr.io/odysseus-dev/odysseus`, multi-arch, published by CI on every push to `main`/`dev`), starts with `docker compose up -d`, and prints the first admin password into the compose logs. Production users are told to pin immutable `X.Y.Z-<sha>` tags rather than the moving `:latest`.

The feature list is unusually broad for a self-hosted project: chat and agents with tools/MCP/files/shell/skills/memory; a "Cookbook" that recommends, downloads, and serves models matched to your hardware; IterResearch-style deep research; blind side-by-side model comparison; an AI-editing document editor; an IMAP/SMTP email inbox with triage and AI drafts; notes/tasks/calendar with CalDAV sync; plus gallery, themes, 2FA.

Security guidance is front and centre in the README: keep `AUTH_ENABLED=true` on anything network-reachable, keep `LOCALHOST_BYPASS=false` outside dev, and don't expose raw model ports. The repo is AGPL-3.0-or-later, packaged in several distro repos (Repology badge), and has a `dev` branch that moves faster than the curated `main`.
