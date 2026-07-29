---
url: https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models
title: The New Rules of Context Engineering for Claude 5 Generation Models
author: Thariq Shihipar
date_fetched: 2026-07-29
date_published: 2026-07-24
---

# The New Rules of Context Engineering for Claude 5 Generation Models

**Author:** Thariq Shihipar, member of technical staff, Anthropic
**Published:** July 24, 2026
**Categories:** Claude Code, Agents
**Products:** Claude Code, Claude Enterprise, Claude Platform

---

The author opens by noting they've previously written about prompting the newest Claude 5 models, but explains that a user's prompt is just one piece of the puzzle. Much of what Claude receives comes from "your system prompt, Skills, CLAUDE.md files, memory, and other sources" — a practice called context engineering. Unlike a one-off prompt, context is reused across many requests and can't be as specific.

A key development: Anthropic "removed over 80% of Claude Code's system prompt for models like Claude Opus 5 and Claude Fable 5 with no measurable loss" on their coding evaluations. The article offers lessons from this experience and points readers to the `/doctor` command in Claude Code (accessible via `claude doctor`) to right-size their configuration.

---

## Unhobbling Claude

The team found they were overconstraining Claude Code across system prompts, CLAUDE.md files, and skills. One example: transcripts showed "several conflicting messages in a single request" — things like "leave documentation as appropriate" clashing with "DO NOT add comments" from different sources.

While Claude can generally infer user intent from overlapping instructions, it must "think more carefully about these overlapping and conflicting messages" before deciding what to do. Constraints that were once necessary to prevent worst-case outcomes have been deletable with newer models, letting context and judgment suffice instead.

Claude Code also has more tools now. Claude once relied on CLAUDE.md as a primary source of memory and guidance; now memory, artifacts, and skills offer alternative ways to "create new ways of loading and sharing context across sessions."

---

## Then and Now

The article identifies several context engineering "myths" that have become outdated.

### Then: Give Claude rules → Now: Let Claude use judgement

Early Claude Code needed strong guardrails — for example, the old system prompt instructed: "default to writing no comments. Never write multi-paragraph docstrings or multi-line comment blocks." For some prompts, though, that guidance was wrong (complex code may need detailed comments, or users may have their own preferences). Newer models handle these nuances. The updated system prompt says: "Write code that reads like the surrounding code: match its comment density, naming, and idiom."

### Then: Give Claude examples → Now: Design interfaces

The old #1 rule for tool usage was providing examples. With the newest models, examples "actually constrains them to a certain exploration space." Instead, think about tool design — what parameters does a tool expose, and how can they be more expressive? The article gives the example of a Todo tool where simply listing status as an enum with values `pending`, `in_progress`, and `completed` hints at proper usage, and a rule about keeping one item in progress defines the expected behavior.

### Then: Put it all upfront → Now: Use progressive disclosure

Claude Code's old system prompt included everything about code review and verification upfront. Now Claude Code uses progressive disclosure — loading the right context at the right time. Verification and code review were moved into skills that Claude can call selectively. Some tools use "deferred loading," meaning the agent must search for their full definitions via ToolSearch before use. The article recommends applying this to your own CLAUDE.md and Skill.md files rather than treating them as a "central repository for every known practice." A tree of files that loads at the right time is preferred.

### Then: Repeat yourself → Now: Simple tool descriptions

Earlier models sometimes needed repeated instructions or responded better to instructions at the end of the context window. That led to duplicating tool instructions in both the system prompt and tool descriptions. The team found they "could delete these repeat examples" and put usage instructions solely in the tool descriptions.

### Then: Memory in CLAUDE.md files → Now: Auto-memory

Users used to be encouraged to manually save items to CLAUDE.md using the `#` hotkey. Now Claude "automatically saves memories that are relevant to the work and to you."

### Then: Simple specs → Now: Rich references

Claude Code's plan mode previously relied on markdown files for plans. While storing plans helped Claude refer to them, the article notes Claude can now handle "increasingly more complicated references" — HTML artifacts, code itself (a spec could be a detailed test suite or a function to port), and rubrics. Rubrics let Claude verify taste in a domain by using "dynamic workflows" with verifier agents.

---

## Applying This to Your Context

### System Prompt
Tied to the product context — it tells Claude what product it's operating in. For Claude Code users, this is rarely modified, but if building your own agent harness, "this is where you should spend a lot of time."

### CLAUDE.md
Keep it lightweight. Briefly describe the repo's purpose, but spend most tokens on "gotchas inside of the codebase" — for example, that types live in one monolithic file only. Avoid stating the obvious that Claude can infer from the file system. Use progressive disclosure: if you have unique verification instructions, create a verification skill and reference it from CLAUDE.md.

### Skills
Think of these as lightweight guides for Claude to find information when needed. Don't overconstrain them except in highly important areas. For long skills, split them into multiple files with progressive disclosure. Skills work best when they encode "particular opinions, knowledge, or best practices" specific to you, your team, or product.

### References
You can `@` mention files as references. These let Claude refer to in-depth information about the current plan — specs, mockups, or entire codebases. Prefer files in code since they provide "clear, high-fidelity instructions to Claude in a language it knows very well." An HTML mockup generally produces better results than a text description or screenshot.

---

## Try Simplifying

The article concludes by encouraging readers to simplify across their system prompts, skills, and CLAUDE.md files — just as Anthropic did. The new `claude doctor` command (or `/doctor` in Claude Code) can help automate this process. For more on prompting advanced models, readers are directed to the companion Fable field guide.
