---
url: https://blog.jonudell.net/2026/06/17/vibe-coding-as-a-team-sport/
title: Vibe Coding as a Team Sport
author: Jon Udell
date_fetched: 2026-06-22
date_published: 2026-06-17
platform: blog.jonudell.net
---

# Vibe Coding as a Team Sport

Jon Udell describes Bram, a desktop tool he is building that wraps a UI around AI coding agents (Claude Code and Codex), anchoring their workflow to git and GitHub. The post argues that structured, team-oriented processes — borrowing from Kasparov's insights about "weak human + machine + better process" outperforming either alone — can bring order to the chaotic "vibe coding" paradigm.

## The Kasparov Parallel

Udell opens by revisiting his earlier post *Working With Intelligent Machines*, quoting the famous chess result where amateur players with three computers and superior process beat grandmasters. This forms the philosophical foundation: process over raw capability.

> "The winner was revealed to be not a grandmaster with a state-of-the-art PC but a pair of amateur American chess players using three computers at the same time."
> "Weak human + machine + better process was superior to a strong computer alone"

## Bram's UI Companion

The tool presents a three-panel layout: a terminal running Claude Code or Codex (left), a companion UI (bottom right), and the app under development (top right). Key UI features include:

- **Voice input via Whisper** — Udell says this is "transformative" for reducing keystroke load due to repetitive stress.
- **Readable agent responses** — the UI echoes agent output more clearly than the terminal.
- **Screenshot pasting** — images display visually rather than as `[Image #1]`.
- **Tool call history** — compact linked list of recent tool uses.

## The Worklist & Approval Gates

Bram introduces a two-gate workflow: **To-Apply** and **To-Commit**. Each item (a plan document with "before" and "after" sections) can be **Approved**, **Iterated**, or **Dropped**. Items stay out of tracked files until the user approves at the To-Apply gate. The plan documents list options considered, justifications, prior art, and verification steps.

## Flexible Workflow

Users can:
- Capture tangents as new worklist items or GitHub issues.
- Promote worklist items to GitHub issues for later recall.
- Skip the worklist entirely for small changes.
- Attach screenshots during iteration cycles.

## Multi-Agent Teamwork

Udell emphasizes switching between Claude Code and Codex, asking one to review the other's work. Agents are encouraged to introduce themselves (e.g., "This is Jon's Codex speaking"). Information that was previously hidden in agent-specific files becomes visible and searchable across the whole team. This extends to human collaborators and their agents too.

## "Just Enough Ceremony"

Udell frames the workflow as providing structure proportional to task size. Small tweaks skip the worklist; larger changes get more formal planning. He notes that **git and gh** command-line tools — "byzantine and cumbersome" for newcomers — become accessible when agents handle the syntax.

> "With Bram we feel we are bringing order to the chaos of vibe coding"

## Links Referenced

1. [Working With Intelligent Machines](https://blog.jonudell.net/2021/07/11/working-with-intelligent-machines/) (2021)
2. [Bram on GitHub](https://github.com/judell/bram)
3. [Series of posts on working with LLMs](https://jonudell.info/newstack/archive.html)
4. [XMLUI](https://xmlui.org) — powers the Bram UI
5. [How to make best use of git and GitHub for AI-assisted software development](https://blog.jonudell.net/2026/06/02/how-to-make-best-use-of-git-and-github-for-ai-assisted-software-development/) (June 2, 2026)
6. [Andrew Schulman's CodeExam tool](https://www.softwarelitigationconsulting.com/what-claude-code-says-about-claude-code-via-work-on-codeexam/)

## Commenter

**Andrew Schulman** (June 18, 2026) — describes himself as "shockingly more productive working with Bram" and points readers to his homepage for a screenshot and description of work on CodeExam.
