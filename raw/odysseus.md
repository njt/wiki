---
url: https://github.com/pewdiepie-archdaemon/odysseus
date_fetched: 2026-07-05
backfilled: true
---

A self-hosted AI workspace for chat, agents, research, documents, email, notes, calendar, and local model workflows.

Quick Start · Setup Guide · Contributing · Roadmap


`dev`is the default branch and gets the newest changes first. Use`main`if you want the more curated branch.

```
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus
cp .env.example .env
docker compose up -d --build
```
Open `http://localhost:7000` when the containers are healthy. The first admin password is printed in `docker compose logs odysseus`.

Native installs, GPU notes, Windows/macOS instructions, HTTPS, and configuration live in the setup guide.

- **Chat + Agents**— local/API models, tools, MCP, files, shell, skills, and memory.
- **Cookbook**— hardware-aware model recommendations, downloads, and serving.
- **Deep Research**— multi-step web research with source reading and report generation.
- **Compare**— blind side-by-side model testing and synthesis.
- **Documents**— writing-first editor with AI edits, suggestions, Markdown, HTML, CSV, and syntax highlighting.
- **Email**— IMAP/SMTP inbox with triage, tags, summaries, reminders, and reply drafts.
- **Notes, Tasks + Calendar**— reminders, todos, scheduled agent tasks, and CalDAV sync.
- **Extras**— gallery/image editor, themes, uploads, web search, presets, sessions, and 2FA.

A full hover-to-play tour lives on the landing page: `docs/index.html`.

Help is welcome. The best entry points are fresh-install testing, provider setup bugs, mobile/editor polish, docs, and small focused refactors. See CONTRIBUTING.md and ROADMAP.md.

Odysseus is a self-hosted workspace with powerful local tools. Keep auth enabled, keep private data out of Git, and do not expose raw model/service ports publicly. Deployment details are in the setup guide.

AGPL-3.0-or-later -- see LICENSE and ACKNOWLEDGMENTS.md.
