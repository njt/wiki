---
url: https://jxnl.github.io/blog/writing/2026/05/10/codex-maxxing/
title: "Codex-maxxing"
author: Jason Liu
date_fetched: 2026-05-18
date_published: 2026-05-10
tags: [codex, agent-workflow, heartbeats, memory, voice, browser-automation]
topics:
  - agent-coding-workflow
---

# Codex-maxxing

Jason Liu describes how his use of coding agents shifted from purely software tasks toward broader knowledge work around November 2024. He found that Codex's latest upgrades made this wider mode feel native, giving his "work somewhere to live." The key behavioral shift came from learning to give work an operating loop.

## Durable Threads

Liu keeps a single pinned thread per important workstream, compacting them for months. He notes these "megathreads" accumulate "history, preferences, and old decisions that I do not want to recreate every time I come back." The tradeoff is cost—long threads may fall out of cache—but for important workstreams, "continuity is worth it."

## Voice Input

The real benefit isn't speed but getting "the unedited version of my thinking." Liu uses Codex's built-in voice alongside Wispr Flow for system-wide dictation. He gives an example: saying something like "I think there is some guy named Ben in Slack who mentioned this" is natural to say but annoying to type. He also feeds transcripts from calls or in-person conversations (via Granola) into the agent.

## Steering

This feature lets you inject the next message after a tool call—"I do not need to wait for each step to finish before deciding the next one." He can keep adding intent while the agent works, then walk away with the queue shaped. This, combined with Heartbeats, makes the "unit of work" become "a small operating loop."

## Memory

Thread memory is trapped unless serialized durably. Liu maintains an Obsidian vault (structured as a GitHub repo) with directories for people, projects, agent files, and notes. An AGENTS.md file instructs the agent to update relevant pages as it learns. He emphasizes that "files force the agent to compress experience into a form that can survive the thread." The repo-as-vault also gives him diffs as a review surface for memory. He distinguishes between Codex's built-in Memories (good for stable preferences) and his checked-in vault (a replacement for "evergreen threads" that "quietly accumulate vibes").

## Computer & Browser Use

Liu distinguishes three modes:
- $browser — for local web surfaces to inspect/annotate
- @chrome — for signed-in sessions with multiple tabs
- @computer — for desktop apps only available as GUIs

He uses connectors ($slack, $gmail, $calendar) because "Slack threads, inboxes, and calendars are where a lot of work shows up before it ever becomes code." Skills package repeatable workflows via Skill Creator and Skill Installer.

## Remote Control

Codex runs from the machine with files/permissions while Liu checks in from mobile to review, answer, approve, or redirect. "The work no longer has to pause just because I changed locations."

## Heartbeats

Thread-local automations that let a thread schedule itself. Three examples:

1. **Chief of Staff** — Every 30 minutes, checks Slack and Gmail for unanswered messages, researches answers, and drafts replies without sending them. When Liu returns, "replies are often already sitting in drafts."
2. **Monitor for feedback** — Checks every 15 minutes for Slack comments on a video, re-renders in Remotion when feedback arrives, and uses @computer to upload the revised version (because the Slack MCP server couldn't upload files). The loop "crossed tool boundaries."
3. **Get a refund** — After a stolen package, Liu had Codex check every 5 minutes for a customer support agent to join, then switch to 1-minute intervals once they replied. "By the time I got out of the shower, the refund was done."

Many Heartbeats also update his Obsidian vault as explicit memory.

## Goals

Liu distinguishes weak goals ("implement the plan") from strong ones with real success criteria. He describes migrating Python's Rich library into Rust, with the goal being: "it must pass all the unit tests from the original library." He notes, "Ambition without verification is just a wish."

## The Side Panel

This is where "Codex stops being only a chat app and starts becoming the place the work happens." Three functions:
- **Inspect artifacts** — Markdown (commentable), spreadsheets (with formula rendering and cell edits), CSVs, PDFs (especially useful with LaTeX), and slides
- **Operate web surfaces** — an in-app browser the agent controls via JavaScript with $browser, where Liu can leave annotations. He lists five surfaces he uses this way: index.html (the "smallest version is often the best"), Storybook, Remotion Studio, Slidev, and Streamlit. Cites Thariq's post about preferring HTML over Markdown as output format.
- **Review changes** — inspect and comment on what the agent is acting on

## Closing

Liu frames the overall shift: "Not that an agent can write code for me, but that more of my work can keep moving after I leave."
