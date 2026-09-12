---
url: https://elezea.com/2025/12/ai-for-product-management/
title: "How I Use AI for Product Work"
author: Rian van der Merwe (Elezea)
date_fetched: 2026-05-14
date_published: 2025-12-14
topics:
  - ai-product-and-business
---

# How I Use AI for Product Work

Rian van der Merwe's field report on using LLMs as a product management thinking partner. Published December 14, 2025 on Elezea.

## Core Philosophy

The AI should function as a **skeptical sparring partner**, not a ghostwriter. The author still writes their own PRDs, OKRs, and strategy docs. The AI provides background research, challenges weak problem statements, spots missing success criteria, and asks "why?" when reasoning gets vague.

Two principles govern every prompt: **context** (who you are, what you're working on, what "good" looks like) and **constraints** (preventing generic or hallucinated output).

## The Three-Layer Prompt System

A folder structure `llm-prompts/` contains:

1. **System Prompts** (`prompts/pm/`, `prompts/technical/`) — Different prompts for different jobs: general PM sparring, document review, idea stress-testing (a debate framework between optimist and skeptic), and technical understanding for non-engineers.

2. **Personal Context** (`context/`) — Files describing role, experience, communication style, product philosophy, current projects, and team context. Pulled into conversations alongside the relevant system prompt.

3. **Reference Materials** (`reference/`) — Syntax guides, documentation templates, internal style guides so output is usable without reformatting.

The author emphasizes: "the magic isn't in any single prompt—it's in how you combine them."

## Daily Workflow

Uses **Windsurf** as the daily driver, leveraging its `@` mention feature to compose the "assistant" on the fly by combining a system prompt, context files, and the current document.

**Document Review**: Reference the review prompt and product philosophy context. "The model comes back with feedback grounded in my own standards—not generic advice."

**Brainstorming Partner**: Conversational prompt for early-stage thinking, "rehearse my reasoning and get challenged on the weak spots before I'm in front of stakeholders."

**Technical Understanding**: Prompts designed to "explain things without condescension but also without assuming I know the jargon."

## MCP Connection to Real Data

MCP servers connect the AI to internal wikis, documentation sites, code repositories, and APIs. Technical prompts instruct the model to search official documentation first, check internal wikis, examine code when docs are incomplete, and "always cite sources with links so I can verify."

This "turns the AI from a general-purpose assistant into something more like an expert who has access to your company's actual knowledge base."

## Record Keeping

A `work/` folder organized by topic stores feedback and refined thinking. After receiving good critique, the author asks the model to "write a summary of the key issues to a Markdown file I can reference later."

## Lessons Learned

1. **Context files are worth the investment** — Describing who you are and how you work pays off in every conversation.
2. **Push back is a feature, not a bug** — Prompts designed to challenge bad thinking.
3. **Iterate on the prompts** — Regular updates based on what works.
4. **Less context is often more** — Too much can dilute the signal; start minimal and add if needed.

## Follow-up

A follow-up post covers how the system evolved: [How My AI Product "Second Brain" Evolved](https://elezea.com/2025/12/how-my-ai-product-second-brain-evolved/)
