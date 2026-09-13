---
url: https://thorstenball.com/talks/how-i-prompt/
title: "How I Prompt"
author: Thorsten Ball
date_fetched: 2026-09-13
date_published: 2026-07-29
topics:
  - agent-coding-workflow
  - agent-memory-and-context
---

Laracon US 2026 talk (29 Jul) by Thorsten Ball — co-creator of the Amp coding agent — showing the actual prompts behind Amp's features. Credentials first: Amp is a complex, beloved distributed system that is ~99% AI-written; nobody on the team writes much code by hand anymore. His prompting, he insists, is boring: no MCP servers, no frameworks, no custom slash commands, one or two skills. "There is no secret sauce."

The one question he wants you to ask before sending any prompt: **how is the model supposed to know what I mean?** A coding agent is an LLM with a context window, and the information it can act on comes from only a few places — the training data (fixed), the context window (system prompt, tool definitions, skills, messages, tool results, your prompt), and the codebase. If you write "fix the bug with the upload" and that information isn't anywhere in those places, the model won't ask "what bug?" — it will say "You're absolutely right" and do whatever it guesses you meant. His mental model: a senior engineer with total public knowledge is hooded, thrown in a van, sat at a desk with your repo, and handed a note. The note is your prompt. Prompting is writing; the goal is to be understood by this specific reader.

The practice that follows: prompts are mostly *pointers* — at folders, files, docs, Slack-thread screenshots, and auto-generated debug reports — plus the constraints and half-formed intentions in his head that exist nowhere else ("no worktrees", "keep this in the back of your head"). He coins the "one-two punch": first ask the agent to *find* the relevant asset/mechanism, then make the real ask once it's in context; a variant points at a working implementation as "the gold standard" before asking for changes to a broken one. Where hand-typing context is too expensive, he attaches screenshots ("90% of my prompts have screenshots") or uses Amp's auto-generated debug prompts and report IDs.

The endgame, and his favorite way to prompt: move the "how" out of the prompt and into the codebase by littering AGENTS.md files through the repo — agents load them as they enter directories, so "take a screenshot of the storybook" is a complete prompt because the trail already teaches the agent how. The information still has to come from somewhere; the codebase just saves you from being the somewhere.
