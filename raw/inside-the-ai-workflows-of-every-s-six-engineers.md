---
url: https://every.to/source-code/inside-the-ai-workflows-of-every-s-six-engineers
title: "Inside the AI Workflows of Every's Six Engineers"
author: Rhea Purohit
date_fetched: 2026-05-14
date_published: 2025-10-27
publication: Every (Source Code)
---

Rhea Purohit interviews six engineers at Every about their personalized AI stacks. Each engineer runs a different configuration of tools, models, and workflows to build and maintain four AI products (Sparkle, Cora, Spiral, Monologue), a consulting business, and a daily newsletter.

## Yash Poojary — General Manager, Sparkle

- Mac Studio + laptop. Runs Claude Code on one machine, Codex on the other, feeding them identical prompts to compare results.
- Claude Code: "friendly developer" (breaks things down). Codex: "technical developer" (literal, precise).
- Uses Figma MCP so Claude reads design system directly (colors, spacing, components), not screenshots.
- Warp terminal.
- Maintains a "learnings doc" — two lines per push about what he learned, stored in the cloud as rolling memory for AI tools.
- Built AgentWatch, an app that pings him when a Claude Code session finishes so he can run multiple sessions in parallel.
- Structures day around one big task plus smaller background ones. "The problem with CLIs is it's easy to get derailed and lose focus."
- Mornings: focused execution. Afternoons: exploration with new agents, apps, features.

## Kieran Klaassen — General Manager, Cora

- Everything starts with a plan generated in Claude Code using custom agents and workflows.
- Three plan sizes: small (one-shottable), medium (span a few files, include review), large (require manual typing, research, iteration).
- Uses Context 7 MCP (by Upstash) to pull up-to-date, version-specific docs and code examples into prompts.
- Plan → GitHub → "work command" turns plan into coding tasks.
- Claude Code primary. Occasionally Codex or Amp for "nerdier" features.
- Review command runs after work: Claude leads, but also uses Cursor and Charlie.
- Process loops until feature is ready to ship.

## Danny Aziz — General Manager, Spiral

- Primary tool: Droid (Factory's CLI). Runs Anthropic and OpenAI models side by side. ~70% of work here.
- Uses GPT-5 Codex for big feature builds, then Anthropic models for refinement and detail work.
- Talks through implementation plans with GPT-5 Codex, asking about second- and third-order consequences.
- Mostly single monitor or laptop. Adds second desktop only for deep design implementation (Figma side-by-side with build).
- "I don't use Cursor anymore. I haven't opened it in months."
- Main interface: Warp (split screens for fast task-switching). Uses Zed for reviewing plan files and specific code.

## Naveen Naidu — General Manager, Monologue

- Everything starts in Linear. "If it's not in Linear, it doesn't exist." Every ticket links back to its original source.
- Recently migrated from Claude Code to Codex for daily work.
- Two-track execution:
  - Codex Cloud: brainstorms approaches, generates draft PRs (not to merge, but to explore ideas and surface edge cases). Accessible via iOS ChatGPT app or web for async background tasks.
  - Codex CLI: real build, refining plan.md, driving file edits step by step in Ghostty.
- Other tools: Xcode for native macOS, Cursor for backend. MCP integrations with Linear, Figma, and Sentry.
- Review process: Codex /review command for automated bug scanning, manual side-by-side before/after comparison, Sentry error logs check for bug fixes.
- Uses Monologue (speech-to-text app he built) to dictate prompts, write ticket descriptions, and update plans.

## Andrey Galko — Engineering Lead

- Doesn't chase every new tool. Longtime Cursor user who left due to pricing (hit monthly limits in a week).
- Switched to Codex, credits GPT-5's improved code generation.
- Earlier OpenAI models produced "suboptimal code" — technically functional but inconsistent, "lazy." GPT-4.5 and GPT-5 changed that.
- Codex strengths: always good at non-visual logic. With GPT-5-Codex, "it finally got good at the user interface, too."
- "I applaud the people at OpenAI for becoming a real menace to Anthropic's code generation reign."

## Nityesh Agarwal — Engineer, Cora

- Entire stack on a MacBook Air M1. "I'm the kind of developer who doesn't like changing my tools often."
- Primary tool: Claude Code (Max plan) for all AI-assisted coding.
- Spends hours researching codebase and sketching detailed plans with Claude before writing code.
- Stays in a single terminal, giving "100 percent attention to the one thing that Claude is working on."
- Watches Claude's work "like a hawk," finger on the Escape key.
- Has shortened Claude's leash, interrupting mid-process to ask for explanations — reduces hallucinations, sharpens his own skills.
- On Claude dependency: "I realize that I've placed too much of my trust in Anthropic, which leaves me vulnerable." When Claude glitched for two days, alternatives didn't match.
- GitHub as review interface: team reviews PRs, leaves line-by-line comments, Claude Code fetches and reads those comments to make fixes collaboratively.
- Cursor and Warp are "solid nice-to-haves."

## Cross-Cutting Themes

| Theme | Details |
|-------|---------|
| Dual-model strategies | Multiple engineers run Claude and OpenAI models side by side |
| Planning-first approaches | Nearly every engineer emphasizes upfront planning before letting AI write code |
| Guardrails against drift | Several engineers describe structures to avoid getting derailed |
| Tool consolidation | Most have narrowed to 1-2 primary coding agents |
| Personal tool-building | AgentWatch, Monologue built within their stacks |
| Code review remains human-led | AI handles initial passes, humans validate |
| Morning/afternoon separation | Yash splits "build" vs. "discover" time |

## All Tools Mentioned

AI Coding Agents: Claude Code, Codex (OpenAI), Codex Cloud, Codex CLI, Amp, Cursor, Charlie, Droid (Factory)
Terminals/Editors: Warp, Ghostty, Zed, Xcode
MCP Integrations: Figma MCP, Context 7 MCP, Linear MCP, Sentry MCP
Other: Linear, GitHub, Figma, Sentry, AgentWatch, Monologue, Sparkle
Products built by Every: Sparkle, Cora, Spiral, Monologue
