---
title: "Rowboat"
url: https://github.com/rowboatlabs/rowboat
date_fetched: 2026-05-14
section: "Personal Agents"
topics:
  - agent-memory-and-context
  - personal-agents
---

# Rowboat: Open-Source AI Coworker

## Main Purpose
Rowboat is a local-first AI assistant that "turns work into a knowledge graph and acts on it." Connects to email and meeting notes to build persistent context, enabling users to accomplish tasks privately on their own machines.

## Key Features

**Core Capabilities:**
- Generate documents and presentations using accumulated knowledge
- Prepare meeting briefs from historical decisions and conversations
- Create voice memos that automatically update key takeaways
- Track people, companies, or topics through live notes
- Visualize and manually edit the underlying knowledge graph

**Live Notes Feature:**
Users can create self-updating notes by mentioning @rowboat, enabling continuous monitoring of competitors, projects, or individuals across web sources and personal communications.

**Data Architecture:**
Maintains an "Obsidian-compatible vault" of plain Markdown notes with backlinks—a transparent working memory stored entirely on the user's device.

## Integrations
- Gmail and Google Calendar
- Rowboat meeting notes or Fireflies
- Composio.dev product library
- Model Context Protocol (MCP) servers for external tools
- Optional: Deepgram (voice input), ElevenLabs (voice output), Exa (web search)

## Technical Stack
Primarily TypeScript (96.6%), with CSS, JavaScript, Python, and Docker components. 1,613 commits.

## Distinguishing Approach
Rather than reconstructing context on-demand, Rowboat accumulates long-lived knowledge where relationships remain explicit and editable—avoiding proprietary lock-in through local Markdown storage.
