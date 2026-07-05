---
url: https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more
title: "Steering Claude Code: CLAUDE.md files, skills, hooks, rules, subagents and more"
author: Anthropic (Claude Code team)
date_fetched: 2026-07-05
date_published: 2026-06-18
---

Claude Code offers seven distinct methods for delivering instructions: CLAUDE.md files (root and subdirectory), rules, skills, subagents, hooks, output styles, and appended system prompts. Each method differs in when instructions load into context, how they survive compaction, and how much context they consume. The post provides a comparison table and detailed guidance on when to use each.

## The Seven Methods

### CLAUDE.md Files

CLAUDE.md is a markdown file at the project root that loads at session start and persists throughout. Subdirectory CLAUDE.md files (e.g., `app/api/CLAUDE.md`) load on-demand when Claude reads a file under that subdirectory.

On compaction, Claude Code re-reads root CLAUDE.md files. Subdirectory CLAUDE.md files share compaction behavior with path-scoped rules — lost until that subdirectory is touched again.

The article warns that in shared repositories, CLAUDE.md tends to grow as every team appends instructions. "Every line loads into every session for every engineer working in the repo, whether it's relevant to their task or not."

Tip: Keep CLAUDE.md under 200 lines, assign an owner, and review changes like code. "Think of this file as giving Claude an overview of your codebase, or as an index pointing to other files where Claude can find more information as needed."

For monorepos, give each team's directory its own subdirectory CLAUDE.md. Use `claudeMdExcludes` to skip irrelevant files. Organization-wide standards can be deployed via MDM or config management and cannot be excluded.

### Rules

Rules are markdown files in `.claude/rules/` that give Claude specific constraints or conventions. Unscoped rules behave like CLAUDE.md — always loaded. Path-scoped rules use a `paths` field in YAML frontmatter to load only when relevant:

```yaml
---
paths:
  - "src/api/**"
  - "**/*.handler.ts"
---
All API handlers must validate input with Zod before processing.
```

Tip: "Reach for a path scoped rule over a nested CLAUDE.md file when the instruction regards a cross-cutting concern or file that appears in multiple (but not all) corners of the codebase."

### Skills

Skills live in `.claude/skills/` as folders containing instructions, scripts, and resources. Each has a `SKILL.md` file. Only the name and description load at session start; the full body loads when invoked via slash command or auto-match. On compaction, invoked skills are re-injected up to a total budget — oldest drop first.

Tip: "Instructions that are procedural, like deploy workflows, release checklists, or review processes, belong in a skill rather than in CLAUDE.md."

### Subagents

Subagents are markdown files in `.claude/agents/` with YAML frontmatter (name, description, optional model/tool fields) and a system prompt body. Name, description, and tool list load at session start; body loads only when called via the Agent tool.

"The subagent then runs in its own fresh context window, and the only thing that returns to your main session is the subagent's final message (often the aggregated result of many subtasks) plus metadata."

Subagents can nest up to five levels deep. Dynamic workflows can orchestrate tens to hundreds of background agents.

Tip: "Use a subagent when a side task like deep search, a log analysis pass, or a dependency audit would clutter your main conversation with intermediate results you won't reference again." Use a skill when you want to observe and direct each step.

### Hooks

Hooks are user-defined commands, HTTP endpoints, MCP tools, or LLM prompts that fire on specific lifecycle events (file edits, tool calls, session start). Registered in `settings.json`, managed policy settings, or skill/agent frontmatter.

Types: command, HTTP, mcp_tool, prompt, and agent. The first three execute deterministically.

Tip: "Use hooks for anything that should happen deterministically: running linters after edits, posting to Slack on completion, or blocking specific commands before they execute." A `PreToolUse` hook can inspect any tool call and exit with code 2 to deny it.

### Output Styles

Files in `.claude/output-styles/` that inject instructions into the system prompt. Never compacted. Warning: "Changes to the output style will replace the default output style" unless `keep-coding-instructions: true` is set.

Built-in styles: Proactive, Explanatory, Learning.

### Appending the System Prompt

The `append-system-prompt` CLI flag adds instructions to the default role without modifying it. Per-invocation only. Diminishing returns for adherence — the more instructions, the less strictly Claude follows them.

## Key Insights

**"Never do this" doesn't work as a prompt instruction.** Claude will follow it most of the time, but under pressure, in long sessions, ambiguous situations, or due to prompt injection, it can fail. "A real guardrail needs to be deterministic." Use hooks and permissions for enforcement. Managed settings are admin-deployed, cannot be overridden, and are the only way to enforce organization-wide guardrails deterministically.

**Procedures belong in skills, not CLAUDE.md.** CLAUDE.md is for facts Claude should always hold. Deployment runbooks and review checklists belong in `.claude/skills/`.

**Unscoped rules waste tokens.** An unscoped rule is mechanically identical to putting the content in CLAUDE.md: always loaded, always costing tokens.

**Use local files for personal preferences.** All file-based methods have user-level counterparts. Keep project-level files for team-wide conventions.
