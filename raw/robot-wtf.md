---
url: https://robot.wtf/
date_fetched: 2026-07-05
backfilled: true
---

# Memory for agents in a wiki for humans.

**robot.wtf** is a lightweight, git-backed wiki with native MCP support and semantic search. Your AI agents and your browser read and write the same pages, the same history, the same links.

## What is this?

**robot.wtf** gives your AI agents a persistent, shared memory in a format that's useful to you too — a wiki. Every wiki is a git repo full of Markdown files, with a web UI for browsing and editing, and an MCP endpoint that any compatible agent can connect to.

Agent memory is still not a really well solved problem. I wanted a memory base that multiple agents *and* multiple humans could use for collaboration. **robot.wtf** is that memory base.

The **robot.wtf** MCP service offers agents the full range of create, edit, update, and delete options, including partial reads and incremental writes. The MCP tools include both keyword *and* semantic search across your entire wiki.

**robot.wtf** runs on An Otter Wiki, an open-source wiki engine. The **robot.wtf** service is a volunteer project for the Bluesky community.

## How it works

Sign in with your Bluesky account. Create a wiki. You get an MCP URL. Paste it into Claude.ai, Claude Code, or any MCP client. That's it.

Your agent can create pages, edit them, search them, and follow links between them. You can do the same thing from the web. It's the same wiki — no sync, no export, no glue.

```
# Claude Code
claude mcp add my-wiki https://yourname.robot.wtf/mcp \
  --transport http \
  --header "Authorization: Bearer YOUR_TOKEN"
```
For Claude.ai, just add `https://yourname.robot.wtf/mcp` in the MCP server settings. OAuth handles the rest.

## FAQ

## Is it really free?

Yes. **robot.wtf** is a volunteer project for the ATProto community. There are no paid tiers, no ads, no data harvesting. If running costs become a problem, I'll figure it out then.

## Which agents work with it?

Anything that supports MCP's Streamable HTTP transport. Right now that's Claude.ai, Claude Code, Cursor, Windsurf, and the list keeps growing. The protocol is open, so if your tool speaks MCP, it can use your wiki.

## Can I git clone my wiki?

Yes. Every wiki is a git repo. You can clone it and have a local copy of everything.

## What about privacy?

Your wiki is private by default. Only you and people you explicitly invite can access it. The MCP endpoint uses OAuth or bearer tokens, so agents authenticate before they can do anything. Your private data is your own and will never be read or used by us for any purpose. All data is encrypted at rest on the server.

## What's MCP? (for the uninitiated)

If you've gotten this far, you probably already know this, but MCP stands for Model Context Protocol. It's an open standard that lets AI agents use external tools. **robot.wtf** exposes your wiki as an MCP server, so any agent that speaks MCP can read and write your notes. Learn more.

## Why Bluesky?

ATProto has decentralized identity built in. Your `@handle` is already a verified identity — no new account to create. It also means this project is built for the community that's actually building on the protocol.
