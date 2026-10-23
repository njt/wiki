---
url: https://crystl.dev/guides/tmux-for-ai-agents/
title: "tmux for AI Coding Agents"
author: crystl (unattributed)
date_fetched: 2026-10-24
date_published: unknown (references 2026 dates; undated)
topics:
  - agent-coding-workflow
  - developer-tools
---

A practical guide to running AI coding agents (mainly Claude Code) inside tmux: session-per-project naming, window/pane layouts, a `.tmux.conf` tuned for agent TUIs, `send-keys`/`capture-pane` for scripting and reading agents, a status-line monitor that scrapes panes for agent state, tmux-over-SSH, and an honest closing list of what tmux cannot do — which pivots into a pitch for crystl, the author's own agent-aware macOS terminal.

The strongest sections are the working details: `--tmux` requires `--worktree` (and uses iTerm2 panes unless `--tmux=classic`), `send-keys -l` prevents text like "Enter" being sent as keypresses, `capture-pane -J` rejoins wrapped lines so greps don't break at seams, and `escape-time 10` fixes laggy arrow keys inside agent TUIs. The status-line script classifies each pane as working/parked/idle by string-matching rendered output — the guide is candid that it "papers by scraping."

The closing "Where tmux stops" section is the real thesis: tmux has no visibility into which agent is parked, no done-events, brittle screen-scraping, no shared state between sessions, and nothing survives a reboot — because it was built to multiplex shells, not agents. Every limitation maps one-to-one onto a crystl feature. Keep tmux for remote boxes and fragile SSH links; the problem with six agents "stopped being multiplexing."
