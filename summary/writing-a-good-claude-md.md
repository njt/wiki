---
title: "Writing a good CLAUDE.md"
author: "Kyle (@0xblacklight)"
source: https://www.humanlayer.dev/blog/writing-a-good-claude-md
date_published: 2025-11-25
date_fetched: 2026-05-14
type: blog
tags: [claude-code, CLAUDE.md, context-engineering, best-practices]
topics:
  - agent-coding-workflow
---

# Writing a good CLAUDE.md

**Author:** Kyle (@0xblacklight), HumanLayer
**Date:** November 25, 2025
**Reading Time:** < 10 min

The post applies to both CLAUDE.md and AGENTS.md, the open-source equivalent for agents like OpenCode, Zed, Cursor, and Codex.

## Principle: LLMs are (mostly) stateless

The core concept is that "LLMs are stateless functions" with frozen weights. They possess no knowledge about your codebase except what you provide in tokens. Coding agent harnesses require explicit memory management, with CLAUDE.md being the default file included in every conversation.

Three critical implications:
1. Agents have zero codebase knowledge at session start
2. Important information must be communicated each session
3. CLAUDE.md is the preferred communication method

## CLAUDE.md Onboards Claude to Your Codebase

The file should cover three dimensions:

- **WHAT:** Technology stack, project structure, codebase mapping (especially vital for monorepos)
- **WHY:** Project purpose and function of different components
- **HOW:** Operational details—package managers, verification methods, test procedures, compilation steps

The presentation method matters significantly; avoid overwhelming Claude with unnecessary commands.

## Claude Often Ignores CLAUDE.md

Claude Code injects a system reminder stating: "IMPORTANT: this context may or may not be relevant to your tasks." Claude filters content based on perceived relevance to current tasks. Information that isn't universally applicable increases the likelihood of instructions being disregarded. Anthropic likely implemented this because many users include non-broadly-applicable "hotfix" instructions, which actually degraded performance.

## Creating a Good CLAUDE.md File

### Less (Instructions) is More

Research indicates frontier LLMs follow approximately 150-200 instructions with reasonable consistency. Smaller models degrade more rapidly. Claude Code's system prompt already contains approximately 50 instructions, consuming a significant portion of the agent's instruction capacity before any custom guidance.

### File Length & Applicability

Content should be universally applicable since CLAUDE.md appears in every session. Avoid task-specific database schema instructions unrelated to current work. Ideal length is under 300 lines; HumanLayer's root file contains fewer than 60 lines.

### Progressive Disclosure

Implement separate markdown files in directories like agent_docs/ for specific topics:
- building_the_project.md
- running_tests.md
- code_conventions.md
- service_architecture.md
- database_schema.md
- service_communication_patterns.md

Reference these files in CLAUDE.md with brief descriptions. Use file:line references instead of code snippets to point to authoritative sources, preventing outdated information.

### Claude is (Not) an Expensive Linter

Never delegate code style enforcement to LLMs. "Never send an LLM to do a linter's job" — they're comparably expensive and slow versus traditional linters. Style guidelines consume context window space and degrade performance. Leverage in-context learning; agents typically absorb existing patterns from codebase examples.

Use deterministic tools like Biome for formatting. Consider Claude Code Stop hooks to run formatters/linters with error presentation to Claude. Alternatively, create Slash Commands incorporating guidelines pointing to version control changes or git status output.

### Don't Use /init or Auto-Generate

Because CLAUDE.md affects every session, it represents "one of the highest leverage points of the harness." Bad context has cascading effects across research, implementation plans, and code generation. Each line warrants careful consideration.

## Six Key Takeaways

1. CLAUDE.md onboards Claude through WHY, WHAT, and HOW dimensions
2. Minimize instructions while maintaining necessity
3. Maintain conciseness and universal applicability
4. Use Progressive Disclosure for task-specific information
5. Employ linters and formatters; avoid making Claude a style checker
6. Manually craft content — CLAUDE.md is too leveraged for automation
