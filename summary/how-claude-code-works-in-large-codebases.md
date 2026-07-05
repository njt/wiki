---
url: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start
title: "How Claude Code works in large codebases: Best practices and where to start"
author: Anthropic Applied AI team (Alon Krifcher, Charmaine Lee, Chris Concannon, Harsh Patel, Henrique Savelli, Jason Schwartz, Jonah Dueck, Kirby Kohlmorgen; feedback from Amit Navindgi, Zoox)
date_fetched: 2026-05-16
date_published: 2026-05-14
---

# How Claude Code works in large codebases: Best practices and where to start

**Category:** Enterprise AI | **Product:** Claude Code | **Reading Time:** 5 minutes
**Blog Series:** *Claude Code at scale* — a new series on best practices for enterprise-scale deployments

## Executive Summary

The article covers patterns observed from successful large-scale Claude Code deployments across "multi-million-line monorepos, decades-old legacy systems," distributed architectures, and organizations with thousands of developers. It emphasizes that the *harness* around the model matters more than the model itself.

## How Claude Code Navigates Large Codebases

Claude Code "traverses the file system, reads files, uses grep to find exactly what it needs, and follows references across the codebase." It operates locally with no codebase index required.

**Key distinction from RAG-based tools:** RAG-powered tools embed the entire codebase and retrieve chunks at query time, but "those systems can fail because embedding pipelines can't keep up with active engineering teams." Claude Code uses agentic search — "There's no embedding pipeline or centralized index to maintain as thousands of engineers commit new code."

**Tradeoff:** Agentic search "works best when Claude has enough starting context to know where to look." The quality of navigation depends on codebase setup via CLAUDE.md files and skills.

## The Harness Components

The article defines five extension points plus two additional capabilities:

### 1. CLAUDE.md Files
Context files Claude reads automatically every session. Root file for the big picture; subdirectory files for local conventions. Keeping them focused on broadly applicable info "will prevent them from becoming a drag on performance."

### 2. Hooks
Scripts that run at key moments. Most valuable use is **continuous improvement** — a stop hook can "propose CLAUDE.md updates while the context is fresh." Start hooks load team-specific context dynamically.

### 3. Skills
Packaged instructions for specific task types, loaded on demand. They "offload specialized workflows and domain knowledge that would otherwise compete for context space." Can be scoped to specific paths so they only activate in relevant parts of the codebase.

### 4. Plugins
Bundle skills, hooks, and MCP configurations into a single installable package. "Plugin updates can be distributed across the organization through managed marketplaces."

### 5. LSP Integrations
Give Claude "symbol-level precision: it can follow a function call to its definition, trace references across files, and distinguish between identically named functions in different languages." Without LSP, "Claude pattern-matches on text and can land on the wrong symbol."

### 6. MCP Servers
"How Claude connects to internal tools, data sources, and APIs that it can't otherwise reach." Most sophisticated teams built MCP servers exposing structured search as a tool.

### 7. Subagents
Isolated Claude instances with their own context window. "Some teams spin up a read-only subagent to map a subsystem and write findings to a file, then have the main agent edit with the full picture."

### Component Comparison Table

| Component | Loads | Best For | Common Confusion |
|-----------|-------|----------|-----------------|
| CLAUDE.md | Every session | Project conventions, codebase knowledge | Using it for reusable expertise that belongs in a skill |
| Hooks | Triggered by events | Automating behavior, capturing learnings | Using prompts for things that should run automatically |
| Skills | On demand | Reusable expertise | Loading everything into CLAUDE.md instead |
| Plugins | Always available | Distributing setup across org | Letting good setups stay tribal |
| LSP | Always available | Symbol-level navigation, error detection | Assuming it's automatic |
| MCP servers | Always available | Internal tool access | Building MCP connections before basics work |
| Subagents | When invoked | Splitting exploration from editing | Running exploration and editing in same session |

## Three Configuration Patterns

### Pattern 1: Making the Codebase Navigable at Scale

Key practices:
- **Keep CLAUDE.md lean and layered** — root file for "pointers and critical gotchas only; everything else drifts into noise"
- **Initialize in subdirectories** — Claude walks up the tree and loads every CLAUDE.md it finds along the way
- **Scope test/lint commands per subdirectory** — running the full suite for one service change "causes timeouts and wastes context on irrelevant output"
- **Use `.ignore` files** — commit `permissions.deny` rules in `.claude/settings.json` so exclusions are version-controlled
- **Build codebase maps** — a lightweight markdown file at root listing top-level folders with descriptions gives Claude "a table of contents it can scan before opening files"
- **Run LSP servers** so "Claude searches by symbol, not by string" — grep for a common function name returns thousands of matches, but LSP "returns only the references that point to the same symbol"

**Caveat:** Edge cases exist — codebases with "hundreds of thousands of folders and millions of files, or legacy systems on non-git version control" — to be addressed in future articles.

### Pattern 2: Actively Maintaining CLAUDE.md Files as Models Evolve

Instructions written for the current model "can work against a future one." Rules that guided Claude through patterns it used to struggle with may become constraining. "Teams should expect to do a meaningful configuration review every three to six months," and also after major model releases when performance plateaus.

### Pattern 3: Assigning Ownership for Management and Adoption

The fastest rollouts "had a dedicated infrastructure investment before broad access." A small team wired up tooling so Claude already fit workflows on first use.

An emerging role is an **agent manager**: "a hybrid PM/engineer function dedicated to managing the Claude Code ecosystem." Minimum viable version is a **DRI** — one person with ownership over configuration, settings, permissions policy, plugin marketplace, and CLAUDE.md conventions.

For governance in large/regulated organizations: start with "a defined set of approved skills, required code review processes, and limited initial access," then expand as confidence builds. The smoothest deployments establish cross-functional working groups "bringing together engineering, information security, and governance representatives."

## Application Notes

Claude Code is designed around conventional setups where engineers are primary contributors, repos use Git, and code follows standard directory structures. Non-traditional setups (game engines with large binary assets, unconventional version control, non-engineer contributors) "require additional configuration work." Anthropic's Applied AI team works directly with organizations for these cases.

## Key Quotes

- "The most successful Claude Code deployments share a set of recognizable patterns"
- "knowledge will stay tribal and adoption will plateau"
- "good setups can stay tribal"

## Acknowledgements

Special thanks to **Alon Krifcher, Charmaine Lee, Chris Concannon, Harsh Patel, Henrique Savelli, Jason Schwartz, Jonah Dueck**, and **Kirby Kohlmorgen** from Anthropic's Applied AI team, and **Amit Navindgi** at Zoox for feedback.
