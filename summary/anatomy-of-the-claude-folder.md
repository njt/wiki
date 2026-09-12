---
url: https://blog.dailydoseofds.com/p/anatomy-of-the-claude-folder
title: "Anatomy of the .claude/ Folder"
author: Avi Chawla
date_fetched: 2026-05-15
date_published: 2026-03-23
topics:
  - misc
---

# Anatomy of the .claude/ Folder

**Author:** Avi Chawla
**Publication:** Daily Dose of Data Science
**Date:** March 23, 2026

## Article Content

### Two folders, not one
Project-level `.claude/` is committed to git and shared by the team. Global `~/.claude/` holds personal preferences and machine-local state like session history and auto-memory.

### CLAUDE.md: Claude's instruction manual
Loaded directly into the system prompt at session start. Can exist at project root, in `~/.claude/` (global), or inside subdirectories for folder-specific rules. Claude reads and combines all of them.

### What actually belongs in CLAUDE.md
**Do include:** build/test/lint commands, architectural decisions, non-obvious gotchas, import/naming conventions, file/folder structure.
**Don't include:** linter/formatter config, full documentation you can link to, long explanatory theory.

A ~20-line example CLAUDE.md is provided covering commands (npm run dev/test/lint/build), architecture (Express + Prisma), conventions (zod validation, `{ data, error }` return shape), and warnings (test DB setup, strict TypeScript).

### CLAUDE.local.md for personal overrides
Auto-gitignored file for individual preferences that shouldn't land in the shared repo. Read alongside the main CLAUDE.md.

### The rules/ folder: modular instructions that scale
Markdown files inside `.claude/rules/` load automatically alongside CLAUDE.md. Path-scoped rules use YAML frontmatter with a `paths` field to activate only when Claude works in matching directories. Rules without a `paths` field load unconditionally.

### The commands/ folder: custom slash commands
Each `.md` file becomes a slash command (e.g., `review.md` -> `/project:review`). Commands use `!` backtick syntax to run shell commands and embed output. The `$ARGUMENTS` variable passes user-provided text after the command name. Example: `review.md` runs `git diff main...HEAD` and reviews for code quality, security, test coverage, and performance. Project commands in `.claude/commands/` (shared). Personal commands in `~/.claude/commands/` (`/user:command-name`).

### The skills/ folder: reusable workflows on demand
Commands wait for you to invoke them. Skills watch the conversation and activate automatically when the task matches the skill's description. Each skill lives in its own subdirectory with a `SKILL.md` file using YAML frontmatter (name, description, allowed-tools). Skills can bundle supporting files (guides, templates). Personal skills go in `~/.claude/skills/`. **Key distinction:** "Commands are single files. Skills are packages."

### The agents/ folder: specialized subagent personas
Each agent is a markdown file with its own system prompt, tool access, and model preference. When Claude needs a specialist, it spawns the agent in its own isolated context window. The `tools` field restricts capabilities (security auditors only need Read/Grep/Glob). The `model` field lets cheaper models handle focused tasks. Personal agents go in `~/.claude/agents/`.

### settings.json: permissions and project config
Controls allow/deny lists for tools and commands. The `$schema` line enables autocomplete in editors. Items in `allow` run without confirmation. Items in `deny` are blocked entirely. Unlisted items trigger a confirmation prompt. A `.claude/settings.local.json` provides personal overrides (auto-gitignored).

### The global ~/.claude/ folder
- `~/.claude/CLAUDE.md` loads into every session (personal coding principles)
- `~/.claude/projects/` stores session transcripts and auto-memory per project
- `~/.claude/commands/`, `~/.claude/skills/`, `~/.claude/agents/` for personal-use definitions

### The full picture
Complete directory tree shown for both `your-project/` and `~/.claude/` structures, covering all six components: CLAUDE.md, CLAUDE.local.md, commands/, rules/, skills/, agents/, and settings files.

### A practical setup to get started
Five-step progression: (1) Run `/init` to generate starter CLAUDE.md, (2) Add settings.json with allow/deny rules, (3) Create 1-2 commands for frequent workflows, (4) Split rules into `.claude/rules/` as CLAUDE.md grows, (5) Add global `~/.claude/CLAUDE.md` for personal preferences. The author notes this covers "95% of projects."

### The key insight
The folder functions as a protocol defining project identity, purpose, and rules. CLAUDE.md is described as the highest-leverage file; everything else is optimization. The article recommends starting small and treating the configuration "like any other piece of infrastructure."

## Notable Comment
Randy Lutcavich (Mar 24) noted that commands have been merged into skills per the official Claude docs: a file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Skills add optional features like supporting file directories and auto-invocation control.
