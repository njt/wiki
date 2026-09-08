---
url: https://macotron.statico.io/
title: Macotron
author: statico
date_fetched: 2026-09-08
date_published: unknown
---

# Macotron — Raw Ingest

Fetched from https://macotron.statico.io/ on 2026-09-08.

Macotron is a free, open-source macOS tool that gives coding agents control of the Mac through a scriptable host API. It installs via Homebrew (`brew install statico/tap/macotron`), is built in Swift with QuickJS as its scripting runtime, and is currently in beta — the author's 1.0 gate is simply "when I consider it stable."

The design centers on two pieces. First, a catalog of 73 built-in plugins; on first launch you pick which ones to install into `plugins/`, and Macotron writes an `AGENTS.md` next to them so a coding agent can discover the API without being told. Second, a single `macotron.*` host namespace — "Apple-shipped tools only" — through which plugins tile windows, read sensors, and talk to models.

The site is a minimal landing page: a headline, install line, capability sections that are largely empty, and the two substantive paragraphs above. It reads as a project announcement rather than documentation.
