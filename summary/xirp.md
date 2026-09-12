---
url: https://backstage.spotify.com/docs/xirp
title: "Xirp — What Xirp provides"
author: Spotify Backstage
date_fetched: 2026-08-14
date_published: unknown
topics:
  - developer-tools
---

Xirp is a macOS desktop app in Spotify's Backstage ecosystem for running parallel AI coding sessions. It runs Claude Code, Codex, or Gemini in persistent terminal sessions, lets you switch between them without losing state, and gives each task its own Git worktree so agents can work in parallel without touching the same checkout. One app manages terminals, Git changes, files, rules, skills, session status, and layouts.

The optional differentiator is **Spotify Portal**: connecting Portal gives agents access to shared organizational context — Workspace wiki pages, Software Catalog entities, resources, records, members, and prior sessions. A session launched from a catalog entity or Workspace can receive that context through MCP, and completed Workspace sessions can be uploaded for teammates and future agents.

Xirp explicitly does **not** replace your coding agent or source control provider — you authenticate and configure each agent through its native CLI. Portal is not required: Xirp runs standalone for local projects, persistent terminals, Git worktrees, files, rules, skills, and grid view.

Current beta scope: macOS only, three agents (Claude Code, Codex, Gemini), optional Portal connection, and manual session upload for Workspace-launched sessions.
