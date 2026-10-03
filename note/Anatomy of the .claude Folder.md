# Anatomy of the .claude Folder

Avi Chawla's tour of every directory and file in the `.claude/` folder, from CLAUDE.md to agents/, with concrete examples and a five-step setup progression. The best single-page reference for the folder's full structure — what each piece does, where it lives, and whether it's shared or personal.

---

## Key Quotes

> "The `.claude` folder is the control center for how Claude behaves in your project."

Chawla frames the folder as infrastructure, not documentation. This is the right framing. It's closer to a Dockerfile or a Makefile than it is to a README — it governs behavior, not just understanding.

> "there are actually two .claude directories, not one"

The project-level `.claude/` is committed to git and shared by the team. The global `~/.claude/` holds personal preferences and machine-local state. This distinction is obvious once stated but many users conflate the two. The project folder is the team's contract; the global folder is your private preferences.

> "whatever you write in `CLAUDE.md`, Claude will follow"

Technically true but dangerously incomplete. As [[Writing a Good CLAUDE.md]] documents, the harness prefaces CLAUDE.md with "may or may not be relevant to your tasks," and instruction-following degrades uniformly as instruction count increases. CLAUDE.md is a suggestion, not a guarantee. The article's cheerful tone here undersells the reality that [[claude-ctrl]] captures: "An instruction that lives only in model context is not a constraint."

> "Keep `CLAUDE.md` under 200 lines."

Chawla's advice overlaps with but is looser than the 150-200 *instruction* budget in [[Writing a Good CLAUDE.md]]. Lines vs. instructions — the latter is the more rigorous metric, since a single bullet point can encode multiple implicit instructions. But as a rule of thumb, "under 200 lines" is directionally correct and easier to check.

> "Commands are single files. Skills are packages."

The cleanest one-line distinction between the two concepts. Commands are lightweight, invoked-on-demand markdown files. Skills are subdirectories with SKILL.md frontmatter, supporting files, and auto-invocation triggers. A commenter noted that the line has since blurred — commands and skills now both create slash commands — but the packaging distinction holds.

## Key Themes

#claude-code #configuration #tool #reference #claude-md #skills #commands #agents #rules

## Structure Overview

| Directory/File | Location | Shared? | Purpose |
|---|---|---|---|
| `CLAUDE.md` | Project root or `.claude/` | Yes (git) | Core instructions injected into every session |
| `CLAUDE.local.md` | `.claude/` | No (gitignored) | Personal overrides |
| `rules/` | `.claude/rules/` | Yes | Modular instructions, path-scoped via frontmatter |
| `commands/` | `.claude/commands/` | Yes | User-invoked slash commands |
| `skills/` | `.claude/skills/` | Yes | Auto-invoking workflow packages |
| `agents/` | `.claude/agents/` | Yes | Specialized subagent personas |
| `settings.json` | `.claude/` | Yes | Allow/deny permissions |
| `settings.local.json` | `.claude/` | No (gitignored) | Personal permission overrides |

The global `~/.claude/` mirrors this structure for personal-use definitions that load across all projects, plus `projects/` for session transcripts and auto-memory.

## Critical Analysis

**This is the best structural reference for the `.claude/` folder I've seen.** It's comprehensive without being exhausting, concrete without being prescriptive. If you only read one article about the folder layout, make it this one.

**The article is too trusting of CLAUDE.md.** Chawla says "whatever you write in CLAUDE.md, Claude will follow" without qualification. This is actively misleading. As [[Writing a Good CLAUDE.md]] and [[CLAUDE.md (Universal)]] both demonstrate, CLAUDE.md instructions degrade under load, the harness itself hedges their relevance, and mechanical enforcement ([[Pre-Commit Lint Checks]], [[Feedback Loop is All You Need]]) is the only thing that actually constrains behavior. The article treats CLAUDE.md as a control panel when it's more like a strongly-worded letter.

**The five-step progression is solid.** Run `/init`, add settings.json, create 1-2 commands, split into rules/, add global CLAUDE.md. That's genuinely where most teams should start, and the 95% claim is defensible. The missing step is "add hooks for mechanical enforcement" — but that's arguably the 5% for advanced users.

**The article is already slightly dated.** The comment about commands merging into skills is material — it means the "commands are single files, skills are packages" distinction is more historical than practical. Both now create slash commands; skills just give you more features. This is a fast-moving target.

**The rules/ section is the most underrated part.** Path-scoped rules that load only when Claude works in matching directories is the cleanest answer to the CLAUDE.md bloat problem. Split your monolithic CLAUDE.md into path-scoped rules and you get context-efficient instructions without the degradation tax. [[Scaling LLMs to Larger Codebases]] and [[Writing a Good CLAUDE.md]] both touch on this but Chawla's explanation of the mechanism is clearer.

**What's missing:** No discussion of hooks (PreToolUse, PostToolUse, Stop), which are arguably more important than commands for production use. No mention of MCP servers or how they interact with the folder. And the article doesn't address the priority question: when CLAUDE.md, a rule, a skill, and a setting all conflict, which wins?

For implementation, pair this with [[claude-code-config (Trail of Bits)]] (the reference settings.json), [[Writing a Good CLAUDE.md]] (the instruction budget), and [[How Intercom Uses Claude Code]] (the enterprise-scale deployment).

---
*Sources: [[raw/anatomy-of-the-claude-folder]]*
*Last updated: 2026-05-15*
