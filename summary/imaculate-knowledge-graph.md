---
url: https://harper.blog/2026/03/11/2026-immaculate-knowledge-graph/
title: "My now immaculate knowledge graph of life"
author: Harper Reed
date_fetched: 2026-05-15
date_published: 2026-03-11
topics:
  - agent-memory-and-context
---

# My now immaculate knowledge graph of life

Harper Reed describes building a personal knowledge graph by processing hundreds of meeting transcripts from Granola (a meeting note tool) into Obsidian, using Claude Code as an AI assistant. The inspiration came from "Botwick," an AI friend who built a network visualization of connections from a collection of decades of notes.

## Extraction Process

Reed checked his Obsidian vault and found his last note was from 2022 with only one word: "hungry." However, he had roughly 600 meeting transcripts from Granola dating back to April 2024. He piped these through a Claude Code skill to generate a knowledge graph.

## The Result

Nodes in the graph represent people and concepts extracted from meetings; edges represent co-occurrence within the same meeting. Notable nodes include the RAND Graduate School, John Borthwick, Jesse Vincent, and James Cham.

## How-To Steps (5 steps)

1. **Stop over-optimizing organization** — Citing Steph Ango's post on Obsidian usage, Reed advises: "Laziness is key. Just make it work for you."
2. **Use an interface that feels conversational** — Something more like Claude than a traditional file browser.
3. **Get transcripts onto disk** — Reed built **muesli**, a Rust CLI (available via `cargo install --git https://github.com/harperreed/muesli.git --all-features`), to extract Granola transcripts locally.
4. **Parse transcripts into an Obsidian-friendly format** — Use the meeting summarization skill at github.com/2389-research/summarize-meetings, or build your own. Output should include meeting summaries, extracted people, extracted concepts, `[[wiki-style links]]`, and clean markdown.
5. **Let it churn** — Transcripts get exported, parsed, and notes are "automagically generated into your Obsidian vault." The graph builds itself.

## Key Point

Reed notes the process "could work with any transcript source, not just Granola" — or even non-textual assets like images or videos.

## Closing

He invites interested readers to contact him at harper@2389.ai. The post was "written 98% by a human."
