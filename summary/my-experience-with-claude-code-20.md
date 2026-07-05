---
url: https://sankalp.bearblog.dev/my-experience-with-claude-code-20-and-how-to-get-better-at-using-coding-agents/
title: "A Guide to Claude Code 2.0 and getting better at using coding agents"
author: Sankalp
date_fetched: 2026-05-15
date_published: 2025-12-27
---

# A Guide to Claude Code 2.0 and getting better at using coding agents

Author: Sankalp (sankalp.bearblog.dev)
Published: December 27, 2025

## Full Content

Sankalp's deep-dive guide to Claude Code 2.0 features, his personal workflow, and the emerging discipline of context engineering. Covers the evolution from Claude Code 1.0 to 2.0, sub-agents, commands, checkpointing, skills, hooks, system reminders, and practical advice for getting better at using coding agents.

### Intro

Opens with Karpathy's description of Claude Code: "it's a little spirit/ghost that 'lives' on your computer." The author notes that release cycles are so fast he's "stopped looking forward to new releases because they just keep happening anyways."

### Why I wrote this post

Two motivations: (1) share his love/hate relationship with Anthropic and why he returned to Claude, (2) help readers keep up with AI coding tools. Three components to augment yourself: stay updated with tooling, upskill in your domain, play more and have an open mind.

"The map is not the territory" — the post shows possibilities and thought processes, not rigid prescriptions.

### Lore — Love/Hate with Anthropic

Timeline of his coding agent journey: Codex era (GPT-5.1-Codex was "overall better despite being slow"), Anthropic's "redemption arc" with Opus 4.5. Opus 4.5 "was roughly at same code-gen capability with GPT-5.1-Codex-Max" but faster and a better communicator. Opus 4.5 "has soul" — superior intent detection. Current stack: Claude Code with Opus 4.5 for execution, Codex with GPT-5.2-Codex-Max for review ("this dynamic has been pretty constant for me for probably a year"). Sonnet 4.5 produced "a lot of slop" and "haphazard changes which would lead to bugs."

### The Evolution of Claude Code

Quality of life improvements in CC 2.0: Sub-agents, checkpointing (Esc+Esc or /rewind), syntax highlighting, LSP support, fuzzy file search, background agents, prompt suggestions with cross-project history search (Ctrl+R).

### Feature Deep Dive

**Commands**: Slash commands (built-in or custom) that append prompts to current context.

**Sub-agents**: Five types — general-purpose, Explore, Plan, claude-code-guide, statusline-setup. Spawned via a Task tool.

**Do sub-agents inherit context?** YES for general-purpose and Plan. NO for Explore — it starts fresh. This is a critical design detail: the Explore agent is a read-only file search specialist "strictly prohibited from creating or modifying files." The author finds Explore summaries are "lossy compression" and prefers having Opus 4.5 read relevant files directly for better attention and cross-referencing.

**How sub-agents spawn**: Via the Task tool, with optional background execution via `run_in_background`.

### My Workflow

**Setup**: CLAUDE.md in all repos (kept short with "pointers to READMEs"), git worktrees, hook that clears CLAUDE.md after 1,000 lines.

**Exploration and Execution**: "Throw-away first draft" — create branch, let Claude write end-to-end, compare vs mental model, then iterate with sharper prompts. "You can spend more time on taste refinement."

**What he uses (and doesn't)**: Claude Code + Opus 4.5 for execution. Codex + GPT-5.2-Codex-Max for review and difficult tasks. Cursor for reading code and manual edits (never cancelled subscription). Also tracks OpenCode, Amp CLI, Vibe CLI, Gemini CLI as alternatives.

**Review**: "probably the most important thing" — uses Codex for review because different models catch different things.

### Context Engineering

"Agents are token guzzlers" — a typical CC session runs 200K-500K tokens. "Effective context windows are probably 50-60% or even lesser" due to attention degradation.

**Context engineering definition**: "the art and science of curating what will go into the limited context window" and "answering 'what configuration of context is most likely to generate our model's desired behavior?'"

**MCP and code execution**: Tool definitions bloat context. The solution: "expose code APIs rather than tool call definitions" with a sandboxed filesystem.

**System reminders**: Tags injected into user messages and tool results to combat context degradation by reciting objectives. Borrowed from Manus's approach of "attention manipulation through recitation" — constantly rewriting todo lists/plans pushes global objectives into recent attention span.

**Skills**: Folders containing SKILL.md + code scripts. Loaded on-demand "like Neo in The Matrix (1999)." They solve prompt bloat by loading domain expertise only when needed rather than keeping it in the system prompt.

**Hooks**: Bash scripts running at lifecycle stages (Stop, UserPromptSubmit, etc.).

**Combining Hooks, Skills, and Reminders**: The whole system is "heavily engineered" so "your task is mainly to use your judgement and prompt it in right direction." "Knowing how things work can help you steer the models better."

### Conclusion

"Don't start a complicated task when you are half-way in the conversation" — the context has already degraded. Start fresh for complex work.

### References

Links to Claude Code docs, OpenCode, Amp CLI, Vibe CLI, Simon Willison's blog, Manus, and others.
