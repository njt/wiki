---
url: https://github.com/prime-radiant-inc/toolsmith
title: "toolsmith — stop agents writing throwaway helper scripts"
author: Jesse Vincent (Prime Radiant)
date_fetched: 2026-11-20
date_published: 2026-11-05
topics:
  - claude-code
  - coding-agents-and-frameworks
---

Toolsmith is a Claude Code plugin (by Jesse Vincent / Prime Radiant, the Superpowers lineage) that stops coding agents from writing throwaway helper scripts — log trawlers, data converters, API wrappers — that get saved wherever the agent happens to be and never found again. Instead, a working agent first searches a shared catalog; if nothing fits, it calls the `toolsmith:toolsmith` subagent, which builds or extends a reusable command-line tool in a git-backed tools repo and replies with the exact command to run.

The mechanism is deliberately asymmetric. Two hooks (SessionStart and SubagentStart) inject a few lines into every agent's context: the definition of a tool versus "mission work," the literal grep command and absolute catalog paths to search, and the instruction to never write such a tool yourself. The `writing-tools` skill carries the full rules; the toolsmith subagent owns the repo at `~/.local/share/toolsmith/tools` (global `bin/` plus per-project `projects/<key>/` scopes) and never touches the caller's project or runs a tool's mutating action on the caller's data — anything side-effectful ships with a `--dry-run` proven on a fixture first.

The load-bearing component is `bin/toolsmith-repo`, a 909-line bash script that owns all git and file plumbing: staging tools into a temp dir, optimistic-concurrency install (base blob hash checked under a repo lock), catalog regeneration from `toolsmith-v1:` headers in each tool's first five lines, and a detailed exit-code protocol. The toolsmith agent never runs git itself, because neither `cd` nor shell variables survive across its Bash calls — a raw `git -C "$TOOLS"` with an unset variable lands in the user's project.

Beyond Claude Code, generated per-harness adapters (via [[Everyharness]]) carry the same repo, catalogs and reply format to Codex, Cursor, Gemini CLI, Pi and nine others, with the skill telling non-hook harnesses how to launch a general-purpose subagent with the full prompt pasted in. Pressure scenarios run real `claude -p` sessions; 18 of 18 trials passed in auto permissions mode.
