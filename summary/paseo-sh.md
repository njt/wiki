---
url: https://paseo.sh/
title: "Paseo"
author: Mo (maintainer)
date_fetched: 2026-10-11
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - developer-tools
---

Paseo is a cross-platform desktop/web/mobile client that unifies every coding agent behind one interface. It is Apache 2.0, self-funded, and run by a single maintainer ("Mo") who has turned down venture funding to keep the tool neutral.

The pitch is threefold. First, **unification**: it wraps multiple agent providers (including adding your own via a plugin) so you can switch between them without changing tools, and brings your existing subscriptions, skills and configuration along. Second, **composability**: agents run locally, on a server, or for a team, and you connect from any device — the FAQ fields questions about connecting from outside the network and whether code leaves the machine (it doesn't, per the privacy stance). Third, **automation**: MCP, a CLI, and a TypeScript SDK expose the same capabilities the UI uses, with a demo of fanning ready GitHub issues out to separate git-worktree agents and reporting back when each finishes.

Extensibility is a first-class axis: plugins can add UI (screens, workspace panels, sidebar items, slash commands, themes), hook the agent lifecycle (configuration, environment, MCP servers, answering permission prompts, follow-ups when a turn ends), and register new providers. The stated philosophy: private (no telemetry, no forced login), independent (no investors), polished (no daemon knowledge required), unified, composable, extensible. Funded by sponsorship and "Paseo Hub"; claims tens of thousands of daily users and one full-time maintainer.
