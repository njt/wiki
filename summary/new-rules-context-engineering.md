---
url: https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models
title: "The New Rules of Context Engineering for Claude 5 Generation Models"
author: Thariq Shihipar
date_fetched: 2026-07-29
date_published: 2026-07-24
---

Thariq Shihipar (Anthropic) argues that prompting is only one part of what Claude sees — the rest comes from system prompts, Skills, CLAUDE.md files, memory, and other sources collectively called "context engineering." The article draws lessons from Anthropic's work simplifying Claude Code's own context for Claude 5 generation models, where they removed over 80% of the system prompt with no measurable loss on coding evaluations.

Six "then and now" shifts define the new approach. **Rules → judgment:** instead of hard constraints ("never write comments"), let the model match the surrounding code's style. **Examples → interface design:** well-designed tool schemas (enums, descriptive parameters) guide behavior better than explicit examples, which can constrain exploration. **Everything upfront → progressive disclosure:** load context only when needed — Claude Code moved verification and code review into skills the model calls selectively, and some tools defer their full definitions behind ToolSearch. **Repetition → simple tool descriptions:** duplicate instructions across system prompts and tool descriptions proved unnecessary for Claude 5 models. **Manual memory → auto-memory:** Claude now saves relevant memories automatically rather than requiring users to use the `#` hotkey. **Simple specs → rich references:** plans can now be HTML mockups, test suites, code to port, or rubrics for taste verification — not just markdown files.

For your own context: keep CLAUDE.md lightweight (repo purpose and gotchas, not the obvious); treat skills as opinionated guides loaded on demand; split long skills with progressive disclosure; and prefer code-based references (HTML mockups, specs, test suites) since Claude reads them with high fidelity. The new `/doctor` command in Claude Code helps automate context simplification.
