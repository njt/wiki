---
url: https://www.railly.dev/blog/intent-layer/
title: "Context Engineering Skills: Intent Layer"
author: Railly Hugo
date_fetched: 2026-05-15
date_published: 2026-01-18
---

# Context Engineering Skills: Intent Layer

**Author:** Railly Hugo

**Date Published:** January 18, 2026 · 2 min read

**Tags:** #ai #engineering #claude

---

## Core Argument

Hugo argues that AI coding agents fail on large codebases not because of model limitations, but because they lack the tacit knowledge senior engineers carry. He introduces **context engineering** as a discipline to bridge this gap, and **`/intent-layer`** as a practical tool for implementing it.

## The Problem

Hugo describes watching Claude burn through "40k tokens exploring dead ends" on a large repo. It found "mocked tests, outdated docs, random utilities" but "miss the config file with the actual bug." The agent searched reasonably but in the wrong places.

## Why This Happens

The root cause is that experienced engineers possess a mental map of a codebase — what each folder owns, what breaks if altered, where actual logic lives — and "that map took years to build. Your agents don't have it."

## The Solution: Context Engineering

Hugo defines context engineering as "designing the full information an agent needs to perform reliably," comprising four components:

- System prompts and instructions
- Structured inputs and outputs
- Tools and their definitions
- RAG and memory systems

Intent Layer addresses the first piece: **system prompt infrastructure**.

## What Intent Layer Does

The skill sets up `AGENTS.md` files at folder boundaries — "simple markdown that gives agents the context they can't get from code alone." Each file documents a folder's purpose, what it does *not* own, contract boundaries, and pitfalls.

An example `AGENTS.md` for a payment service covers:
- Purpose (payment processing, *not* billing/invoicing)
- Contracts (processor calls go through a specific client file; settlement config lives in a shared config folder)
- Pitfalls (a legacy folder still handles pre-2023 accounts; test mode charges are real)

Running the skill on a project causes it to:
1. Detect existing CLAUDE.md / AGENTS.md files
2. Analyze codebase structure
3. Suggest where to add context nodes
4. Ask what patterns and pitfalls to document

It can be re-run later to audit or find new candidates as the project grows.

## Results

With AGENTS.md files in place, the same bug resolution consumed only "16k tokens loaded (not 40k)" and the agent "went straight to the config file, found it first try."

## Themes & Concepts

- **Tacit knowledge transfer:** Converting senior engineers' mental maps into explicit, agent-readable documentation
- **Hierarchical context nodes:** AGENTS.md files at folder boundaries that cascade into a layered understanding
- **Token efficiency:** Proper context reduces wasted inference cost by more than half in the example given
- **Iterative auditing:** The tool can be re-run as codebases evolve, treating context documentation as a living artifact
- **Open source availability:** All skills published at [crafter-station/skills](https://github.com/crafter-station/skills)

## Credits

Hugo credits Tyler Brandt's "The Intent Layer" and his "AI Adoption Roadmap" as foundational. The context engineering framework draws from DAIR.AI and LangChain. This is the evolved version of his earlier "LLMS.md" idea from his "AI-First Manifesto."

## Install Command

`npx skills add crafter-station/skills --skill intent-layer -g` (works with Claude Code, Codex, Cursor, Copilot, and 10+ other agents)
