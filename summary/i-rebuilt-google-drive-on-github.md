---
url: https://adventuresinclaude.ai/posts/i-rebuilt-google-drive-on-github
title: "I Rebuilt Google Drive on GitHub"
author: adventuresinclaude.ai
date_fetched: 2026-11-17
date_published: 2026-11
topics:
  - personal-agents
  - agent-coding-workflow
---

# I Rebuilt Google Drive on GitHub

The author spent a year trying to make Claude Code work with Google Drive, gave up on the friction, and rebuilt the whole thing with GitHub as the storage layer — a "Doc System": drafts in a private git repo, auto-committed every Claude turn, published by an automated pipeline straight to `main` on a protected branch, and surfaced through an internal web app (IntensityOS) that mimics Drive's folders, tags, sharing, comments, and suggestions.

Four design choices differentiate it from Drive: agents participate under their own identities (his laptop agent "Fluffy" can search, read comments, suggest edits, close threads, and publish, with `/co-pushdoc revise` handling comment threads end to end); every version of every document is a git commit on a protected branch; files are HTML/Markdown, Claude's native formats, so there is no conversion step; and the agent works through `git`/`gh` with no special API — the web app exists only for humans. Safety rails: a never-publish block list for personal/health/people documents, and gitleaks scanning before publication.

He surveys the open-source neighborhood (Wiki.js, Gollum, Outline, Docmost, Nextcloud, Obsidian+git) and finds none combine git as the document home, a real collaboration layer, and agents acting under their own names. The post ends unresolved: the code is easy to open source, but he doesn't want the maintainer's life, and asks readers who've cracked that problem to write in. The closing irony: the publisher can export to Google Drive as PDFs and Google Docs — "Drive works fine as an output. It just doesn't work as the place where the work happens."
