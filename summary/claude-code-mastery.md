---
url: https://arps18.github.io/posts/claude-code-mastery/
title: "Beyond the Prompt: Claude Code"
author: Arpan Patel
date_fetched: 2026-06-05
date_published: 2026-05-26
tags:
  - claude-code
  - skills
  - subagents
  - CLAUDE.md
  - MCP
  - workflow
  - devtools
topics:
  - agent-coding-workflow
---

# Beyond the Prompt: Claude Code

By Arpan Patel · May 26, 2026 · 27 min read

## Summary (via WebFetch)

A comprehensive guide to mastering Claude Code beyond basic prompting. Covers `.claude` directory structure, CLAUDE.md best practices from Boris Cherny, skills as reusable expertise units, custom subagents, plugins and the marketplace, underused commands (`/goal`, `/compact`, `/rewind`), MCP integration, daily workflow optimization, and tips directly from the Anthropic team.

## 1. Claude Code Beyond the Basics

The core philosophy — attributed to Boris Cherny and the Anthropic team — is to "give Claude a way to verify its own work." Without that verification loop, the user is the only feedback signal; with it, Claude iterates until code actually runs. This is pegged as delivering a 2-3x quality improvement.

**Key patterns:**

- **Explore, then plan, then code:** Hit `Shift+Tab` twice for read-only plan mode. Skip planning for small fixes; use it when changes span multiple files.
- **Treat plan mode like a design doc:** Have one Claude write a plan, then a second Claude in a fresh session reviews it as a staff engineer.
- **Reference, don't describe:** Use `@src/auth/login.py` instead of describing files. Pipe errors with `cat error.log | claude`.
- **Delegate, don't pair-program:** Cat Wu (Claude Code team) states: "The model performs best if you treat it like an engineer you're delegating to, not a pair programmer."

**Tips:** Press `Ctrl+G` to open Claude's plan in your editor for tweaking. End prompts with "Update CLAUDE.md so you don't repeat this" after mistakes — Boris calls Claude "eerily good at writing rules for itself."

## 2. The .claude Directory, Properly Understood

Two scopes: **Project scope** (`.claude/` inside the repo, committed) and **Global scope** (`~/.claude/`, rides along on every project).

**Mental model:** Project files describe the project; global files describe you.

| File | Scope | Commit | Purpose |
|---|---|---|---|
| `CLAUDE.md` | Project & global | Yes | Instructions loaded every session |
| `CLAUDE.local.md` | Project only | No (gitignore it) | Private project notes |
| `settings.json` | Project & global | Yes | Permissions, hooks, env vars, model defaults |
| `settings.local.json` | Project only | No | Personal overrides, auto-gitignored |
| `.mcp.json` | Project only | Yes | Team-shared MCP servers |
| `skills/<name>/SKILL.md` | Project & global | Yes | Reusable prompts invoked with `/name` |
| `commands/*.md` | Project & global | Yes | Single-file slash commands |
| `agents/*.md` | Project & global | Yes | Subagent definitions |
| `rules/*.md` | Project & global | Yes | Topic-scoped instructions, optionally path-gated |

**Easy misses:** CLAUDE.md files cascade in monorepos. Rules are path-gated via globs. Skills beat commands because skills carry supporting files, tool restrictions, and agent overrides.

## 3. CLAUDE.md, The Way Boris Writes It

Two principles from Boris:

- **Keep it short.** For every line, ask: "Would removing this cause Claude to make a mistake?" If no, cut it.
- **Let Claude write rules for itself.** After mistakes, tell it to update CLAUDE.md. Over weeks, this becomes a curated list of project gotchas.

### 3.1 The Real CLAUDE.md From the Claude Code Team

```
# Development Workflow
**Always use `bun`, not `npm`.**
# 1. Make changes
# 2. Typecheck (fast)
bun run typecheck
# 3. Run tests
bun run test -- -t "test name"
bun run test:file -- "glob"
# 4. Lint before committing
bun run lint:file -- "file1.ts"
bun run lint
# 5. Before creating PR
bun run lint:claude && bun run test
```

Boris uses Claude in PR comments to have Claude commit rules — calling this "Compounding Engineering."

**Fleshed-out template:**
- Code style (ES modules, not CommonJS)
- Workflow (bun, not npm; typecheck before done; never push to main)
- Architecture (auth middleware, db queries location)
- Gotchas (type distinctions, locale assumptions)

**What to skip:** standard language conventions, file-by-file codebase descriptions, long tutorials, API docs, anything that changes frequently.

## 4. CLAUDE.local.md as a Daily Driver

Sits next to CLAUDE.md, loads the same way, goes into `.gitignore`. After every PR, paste reviewer comments here. Two sections: project-specific feedback and personal habits. Prune after a few weeks.

## 5. Skills, In Depth

Skills transform Claude from "agent that can do anything" to "agent that does the three specific things your project needs, done your team's way."

A folder dropped into `~/.claude/skills/` with a `SKILL.md` containing frontmatter and instructions. The folder name becomes the slash command.

**Three properties:**
1. **Progressive disclosure** — only ~100 tokens of frontmatter loaded at session start; full content loads when invoked
2. **Each skill lives as its own folder** — can include templates, reference docs, scripts
3. **Inline shell** — `!` runs a command at invocation time and splices output into the prompt

**Frontmatter knobs:** `name`, `description`, `disable-model-invocation`, `allowed-tools`, `agent`.

**Popular skills:**
- mattpocock/skills (~100k stars): `/grill-me`, `/tdd`, `/diagnose`
- Jeffallan/claude-skills: 66 language-specific profiles
- Anthropic's official: `/code-review`, `/simplify`, `/batch`, `/webapp-testing`

## 6. Building Custom Subagents

Drop a markdown file into `.claude/agents/` (project) or `~/.claude/agents/` (global). Frontmatter declares name, description, tools, and model.

**Example `/pr-review` agent:** Run `git diff main...HEAD`, read full files, cross-check against CLAUDE.md and rules. Flag correctness bugs, security, missing tests, N+1 queries, convention violations. Output grouped by severity with SHIP/FIX FIRST/REWORK verdict.

**Popular subagents:** security-reviewer, test-writer, debugger, performance-auditor, migration-writer, release-notes-writer.

**Collections:** VoltAgent/awesome-claude-code-subagents (100+ agents), hesreallyhim/a-list-of-claude-code-agents.

## 7. Plugins and the Marketplace

Plugins bundle skills, hooks, subagents, and MCP servers. Run `/plugin` for the marketplace browser. 1,000+ plugins, 75+ marketplaces as of mid-2026.

**Day-one installs:** `/code-review`, `/feature-dev`, language server plugin, `/security-guidance`.

## 8. Underused Claude Code Commands

| Command | Purpose |
|---|---|
| `/insights` | Analyzes usage patterns |
| `/compact <hint>` | Compresses session; hint controls what survives |
| `/copy` | Copies last response with interactive picker |
| `/rewind` | Undo for session, restoring code and/or conversation |
| `/btw` | Side question that never enters history |
| `/context` | Visualizes context usage |
| `/export <file>` | Dumps conversation to file |
| `/branch` | Forks session to try something risky |
| `/batch` | Fans work to parallel agents across worktrees |
| `/loop <interval>` | Schedules Claude on repeat, up to 3 days |
| `/schedule` | Cloud version of `/loop` |
| `/teleport` | Moves session between terminal and web |
| `/focus` | Hides intermediate tool calls, shows only final result |
| `/voice` | Voice input |
| `--bare` | Up to 10x faster startup for non-interactive `claude -p` |

### 8.1 /goal, the Ralph Loop Built In

Sets a completion condition; Claude keeps working until it holds true.

Example: `/goal all tests in test/auth pass and the lint step is clean`

Real examples from the article include integration tests passing without flaking 3 runs in a row, OpenAPI spec validation, docker compose health checks, and coverage thresholds.

**Tip:** "Combine `/goal` + auto mode + `/focus`. Write a crisp brief, set the goal, walk away."

## 9. MCPs as Power Tools

MCP (Model Context Protocol) turns Claude Code into a system-aware agent.

**Go-to MCPs:** GitHub, Context7, Sentry, Linear, Playwright, Figma, Postgres/Supabase, Slack.

**Connection:** Local servers use stdio; vendor-hosted use HTTP with OAuth.

**Obsidian workflow:** Install `obsidian-claude-code-mcp`, drop a CLAUDE.md at the vault root. Three tiers of memory: hot (daily session logs), warm (project notes + recent logs), cold (decisions and atoms via wikilinks).

**Tip:** Resist installing every MCP — bloated tool lists hurt decision quality.

## 10. Optimizing Your Daily Workflow

- **Morning:** Skim overnight output. Run `/insights` weekly.
- **New feature:** Plan mode → `Ctrl+G` → implement → pr-review or fresh session review.
- **Bug:** Reproduce first. Pipe error. Failing test before fix.
- **Migrations:** Use `/batch` — fans out to parallel agents in worktrees, each opens its own PR.
- **Unfamiliar code:** Hand to subagent. Keep main session clean.
- **Parallel sessions:** 3-5 git worktrees, each running its own Claude session.
- **Writer/Reviewer pattern:** Session A implements, Session B reviews fresh. Repeat until Session B stops complaining.
- **Compact at milestones:** `/compact Preserve the decisions made, files changed, and test commands.`

**Tip:** "Never let Claude claim success without evidence, whether that's tests, screenshots, or real command output."

## 11. Tips From the Anthropic Team

- "Give Claude a way to verify its output."
- Use Opus with high or xhigh effort for almost everything
- Run 3-5 sessions in parallel (worktrees beat checkouts)
- Keep a notes directory per project, updated after every PR
- Build a `/techdebt` slash command, run at end of every session
- Team CLAUDE.md is shared and edited multiple times a week
- `Esc` twice opens rewind; pair with checkpoints
- Set up Playwright MCP for UI changes
- Install a language server plugin
- Use `/voice` for prompting (speak 3x faster than typing)
- Auto mode + `/focus` + `/goal` — write brief, set goal, walk away
- Use `Ctrl+G` to edit Claude's plan before implementation
- Ask Claude to draw ASCII diagrams of new protocols and codebases

## 12. Resources

**Official docs:** Claude Code documentation, .claude directory docs, best practices, memory docs, skills, subagents, plugins, MCP, hooks

**Boris and the team:** howborisusesclaudecode.com, Anthropic blog on Opus 4.7 best practices, shanraisshan/claude-code-best-practice

**Skills:** mattpocock/skills, Jeffallan/claude-skills, addyosmani/web-quality-skills, Anthropic skills cookbook

**Subagents:** VoltAgent/awesome-claude-code-subagents, hesreallyhim/a-list-of-claude-code-agents

**Plugins:** Chat2AnyLLM/awesome-claude-plugins, claudemarketplaces.com

**MCPs:** Obsidian Claude Code MCP plugin, official MCP servers list, claude.com/partners/mcp

## Closing Notes

The author's core insight: the mental model flipped from "I need to write this code" to "I need to set Claude up to write this code well." Setup is the work; execution is verification.

**Key takeaways:**
- CLAUDE.md is compounding infrastructure — every mistake becomes a rule
- CLAUDE.local.md captures PR feedback as free training data
- Skills are the unit of reusable expertise (if you've prompted something twice, write a skill)
- Subagents over kitchen-sink prompts — keep contexts clean
- Parallel sessions are the most underestimated unlock
