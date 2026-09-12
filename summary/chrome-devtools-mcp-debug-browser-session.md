---
url: https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session
title: "Let your Coding Agent debug your browser session with Chrome DevTools MCP"
author: Sebastian Benz and Alex Rudenko
date_fetched: 2026-05-15
date_published: 2025-12-11
topics:
  - developer-tools
---

# Let your Coding Agent debug your browser session with Chrome DevTools MCP

The Chrome DevTools team announced an enhancement to their MCP server in Chrome M144 (beta at time of writing) that lets coding agents connect to an *already-running* browser session via a new `--autoConnect` flag. This builds on Chrome's existing remote debugging infrastructure. When an agent requests a connection, Chrome shows a permission dialog; during active sessions, an infobar warns "Chrome is being controlled by automated test software."

## How it works

Three-step setup:
1. Enable remote debugging at `chrome://inspect/#remote-debugging`
2. Configure the MCP server with `--autoConnect` (and `--channel=beta` until M144 hits stable)
3. Prompt your agent — it triggers a Chrome permission dialog, user clicks Allow

The feature is explicitly designed for a hybrid workflow: a developer manually inspects a network request or DOM element in DevTools, then says "hey agent, investigate this." The agent reuses the existing session — already authenticated, already at the right page, already showing the relevant DevTools panel data.

## Key quotes

> "We shipped an enhancement to the Chrome DevTools MCP server that many of our users have been asking for"

> "Seamlessly transition between manual and AI-assisted debugging"

> "You don't have to choose between automation and manual control"

## Technical notes

- The `autoConnect` option requires the user to start Chrome; it doesn't launch the browser — it connects to an existing one
- Remote debugging must be explicitly enabled by the user (off by default)
- The feature builds on existing remote debugging capabilities of Chrome, not new infrastructure
- Planned future: expose more DevTools panel data through MCP

Example config for `gemini-cli`:
```
npx chrome-devtools-mcp@latest --autoConnect --channel=beta
```
