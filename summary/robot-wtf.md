---
title: "robot.wtf"
url: https://robot.wtf/
date_fetched: 2026-05-14
section: "Personal Agents"
topics:
  - agent-memory-and-context
---

# robot.wtf: AI Agent Memory Wiki

## Purpose
robot.wtf provides persistent, shared memory infrastructure for AI agents using a wiki format that humans can also access and edit. It bridges the gap between agent-readable data structures and human-usable documentation.

## Key Features

**Shared Interface**: The same wiki pages, history, and links are accessible to both AI agents and humans. "Your AI agents and your browser read and write the same pages."

**Git-Backed Storage**: Each wiki is "a git repo full of Markdown files," allowing users to clone wikis locally and maintain version control.

**MCP Integration**: The system exposes wikis as Model Context Protocol servers, enabling compatible AI tools to integrate seamlessly.

**Search Capabilities**: Both keyword and semantic search across the entire wiki.

**CRUD Operations**: Agents can create, read, update, and delete wiki content, including partial reads and incremental writes.

## How It Works

1. Users sign in via Bluesky authentication
2. Create a wiki instance
3. Receive an MCP endpoint URL
4. Connect that URL to Claude, Cursor, Windsurf, or other MCP-compatible clients
5. Agents immediately gain read/write access to the wiki

## Privacy & Architecture

Wikis are private by default with explicit invite-only access. OAuth and bearer tokens for authentication. Data encrypted at rest. The project runs on An Otter Wiki, an open-source wiki engine, and operates as a volunteer initiative for the Bluesky/ATProto community.
