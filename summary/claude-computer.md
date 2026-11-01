---
url: https://slop.boutique/apps/claude-computer
title: "Claude.Computer"
author: slop.boutique
date_fetched: 2026-11-02
date_published: ~2026-10 (early access until 8 November 2026)
topics:
  - agent-architecture
  - coding-agents-and-frameworks
---

Claude.Computer (slop.boutique) is a product page for an early-access app that turns a Claude conversation into a shared spatial desktop, one per conversation: instead of answering in chat, Claude builds files and small apps — documents, spreadsheets, slides, canvases, quizzes, custom apps — that the human and the agent both see and both edit. The chat shrinks to a sentence; the work lives on the desktop.

The architecture is the interesting part, and it is deliberately minimal. The whole thing is "one HTML file wrapped in a claude.ai artifact" — every conversation that gets the link publishes its own copy with its own storage. Claude never writes files or events directly; it writes a plain-text block format (`@@doc Name`, `@@slides Name`) that the page parses *while it is still streaming*, so drafts appear where they will land and become real files the moment a block closes. Nothing Claude writes can break the desktop because the desktop is not exposed to it.

Each agent turn is just two calls: read one view document (your changes since last turn, the files, an index, what you're looking at and selecting, the previous turn's outcome), and write one turn document in the block format. The page plays that turn back as Claude working, logs every change so it can be undone, and refreshes the view. Apps split into content and behaviour: built-in apps (docs, sheets, presentations, whiteboards, records, and more) hold content that Claude writes in a few plain lines and that survives Claude's rewrites; *Tanke* apps are small programs that hold behaviour and refer to files by name; *programs* are plain JavaScript that runs on demand and can call Claude via `ask()`.

The page names "three speeds of intelligence" — the app's own logic for instant interaction, a program's `ask()` for small mid-run judgments, and a full Claude turn for anything needing memory, tools or the web — plus a canvas agent that reads the files and your selection and streams its answer straight onto the canvas. Apps can hand the turn back: a "Send to Claude" button makes a click into your next message. The pitch, in its own words: two hands on one desktop, answers are things you use rather than walls of text, and most of the interface is hidden.
