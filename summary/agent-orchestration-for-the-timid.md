---
url: https://markferree.substack.com/p/agent-orchestration-for-the-timid
title: Agent Orchestration for the Timid
author: Mark Ferree
date_fetched: 2026-05-22
date_published: 2026-01-24
platform: Substack (markferree.substack.com)
---

# Agent Orchestration for the Timid

Mark Ferree explores five AI agent orchestration tools after a negative experience with Steve Yegge's Gas Town. He describes himself as a heavy Claude user approaching "stage 8" and expresses skepticism about most orchestration tools, ultimately leaning toward "Claude-centric features for orchestration" rather than adopting external orchestrators.

## Tools Reviewed

### 1. Vibe Kanban
- Install: `npx vibe-kanban`
- URL: https://github.com/BloopAI/vibe-kanban
- Impressions: "less buggy but less capable Agor"
- Runs with `--dangerously-skip-permissions` / `--yolo` by default
- Described as "feels like Jira" — manual task typing and branch creation
- Praised for automatic environment detection and large agent list
- Verdict: Might revisit as an Agor substitute

### 2. Conductor.build
- URL: https://www.conductor.build/
- Impressions: Instantly dismissed due to a "Google form to join the Windows/Linux waitlist"

### 3. Claude Squad
- URL: https://github.com/smtg-ai/claude-squad
- Terminal-only, with tmux integration
- Described as "a friendly Tmux wrapper around standard Claude sessions"
- Learning curve described as steep; author got "frozen staring at the command output"
- Verdict: "Too close to my personal workflow to be useful for me"

### 4. Claude-Flow
- URL: https://github.com/ruvnet/claude-flow
- Uses npx; install described as "slow and full of warnings"
- Author called it a "vibe coded fever dream" category alongside Gastown
- First task had no ID; subsequent commands broken
- Multiple installation methods described as not giving "enterprise vibes"
- Verdict: Set aside to potentially revisit later

### 5. Taskmaster
- MCP support noted; author says MCP "should probably be table stakes for any ai-orchestration tool"
- Uses "Hamsters" as a metaphor instead of polecats
- Clicking Hamster leads to a subscription signup — author calls this "a big nope"
- CLI flags felt like "using the AWS cli (thats not a complement)"
- Verdict: "off the list trying too hard to be commercial"

## Key References
- Gas Town by Steve Yegge: https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04
- Agor (prior orchestration tool the author preferred)
- Beads: https://github.com/mrf/tmux-window-notifier (author removing from projects)
- Taskwarrior — used as analogy for AI-codegen leading to duplication

## Author's Final Position
Ferree plans to "lean harder into Claude-centric features for orchestration than introduce the overhead of any of these orchestrators." He acknowledges Vibe Kanban as the most polished candidate for replacing Agor but ultimately isn't convinced any tool adds enough value over his existing setup of custom skills and slash commands.
