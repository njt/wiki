---
url: https://www.hunk.dev/
title: Hunk — Diff Viewer
author: modem-dev
date_fetched: 2026-08-08
topics:
  - developer-tools
---

Hunk is a terminal-based diff viewer designed for code review in the agent era. It renders an entire changeset as one continuous stream in sidebar order, eliminating the single-file-flipping workflow of traditional diff tools. File counts are visible at a glance, and `[` / `]` keys jump between hunks.

Its headline feature is **agent annotation support**: agents leave review notes in a sidecar file, and Hunk renders each one inside the diff directly above the annotated hunk, with summary, rationale, and author metadata. Reviewers step through hunks with `[` and `]` and the annotations follow.

Layout adapts to terminal width automatically: side-by-side when wide, stacked when narrow. Users can override with `1`, `2`, or `0` at any time. Ships with built-in themes (Catppuccin, Dracula, Gruvbox, GitHub, and others) plus custom theme support via config.

Installation is available through npm (`npm i -g hunkdiff`), Homebrew (`brew install hunk`), or Nix (`nix run github:modem-dev/hunk`). The npm package requires Node.js 18 or newer. Commands: `hunk diff` to review the working tree, `hunk show` to review a commit.
