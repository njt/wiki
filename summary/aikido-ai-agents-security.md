---
url: https://thenewstack.io/aikido-ai-agents-security/
title: "\"There is no accountability\": AI coding agents are installing packages no one owns"
author: Darryl K. Taft
date_fetched: 2026-06-04
date_published: 2026-05-27
publication: The New Stack
tags: security, ai-agents, supply-chain, packages, aikido
---

## Summary

Darryl K. Taft reports on the growing accountability gap where AI coding agents (Claude Code, GitHub Copilot, Cursor, etc.) autonomously install packages and dependencies, but no one in the organization owns responsibility for what gets installed. The article covers Aikido Security's response: three products (Endpoint, Infinite, Intel) aimed at securing the AI agent supply chain. The core insight is that AI agents are spreading beyond developer teams into marketing, sales, and product — expanding the attack surface to people who have no security training and no concept of what they're installing.

## Key People

- **Willem Delbare** — Co-founder, CTO, and CEO of Aikido Security. Source of the accountability-gap framing.
- **Charlie Eriksen** — Aikido researcher who discovered the hallucinated `react-codeshift` package spreading through agent skills (published Jan 21, 2026 on the Aikido blog).

## Key Products

- **Aikido Endpoint** — Inspects packages, plugins, IDE/browser extensions before installation and blocks malware proactively. Covers AI tools including Gemini, OpenAI, GitHub Copilot, xAI, MCP Servers, Claude Code, and skills.sh.
- **Aikido Infinite** — Continuous AI penetration testing platform.
- **Aikido Intel** — Dual LLM models scanning changelogs/release notes for undisclosed vulnerabilities.

## Competitive Landscape

Socket, Endor Labs (AURI), Chainguard, Snyk, Arcjet, and Mobb Security are all working on adjacent pieces of the AI agent security puzzle.

## Related: Hallucinated npx Commands

In a related Aikido blog post (Jan 21, 2026), researcher Charlie Eriksen documented how LLM-generated "Agent Skills" spread hallucinated `npx` commands. The case study: `react-codeshift` — a non-existent package name hallucinated by an LLM (a mashup of real packages `jscodeshift` and `react-codemod`). It appeared in 237+ GitHub repositories via a skill dump in the `wshobson/agents` repo (commit 65e5cb0, Oct 17, 2025), with no human review. Eriksen claimed the package name on npm before any attacker could. After claiming it, download telemetry showed "a persistent trickle of 1-4 downloads per day" — proving real AI agents were executing the hallucinated instructions. The attack surface chain: LLM hallucinates plausible name → skill file embeds it as executable command → agent runs `npx` and hits `y` on the prompt → unclaimed npm name is first-come, first-served.

> "The supply chain just got a new link, made of LLM dreams." — Charlie Eriksen

## Key Quote (from the article)

> "There is no accountability" — Willem Delbare, on AI coding agents installing packages in organizations where no single person or team owns the responsibility for what gets installed.
