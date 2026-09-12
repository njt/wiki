---
url: https://simonwillison.net/2025/Sep/30/designing-agentic-loops/
title: "Designing agentic loops"
author: Simon Willison
date_fetched: 2026-05-14
date_published: 2025-09-30
tags: [definitions, ai, generative-ai, llms, ai-assisted-programming, ai-agents, coding-agents, async-coding-agents]
series: "How I use LLMs and ChatGPT (entry #30)"
topics:
  - agent-coding-workflow
---

# Designing agentic loops

Simon Willison argues that a critical new skill for maximizing LLM-based coding tools is **designing agentic loops** — carefully crafting the tools and iterative processes that coding agents use to brute-force their way to solutions. He defines an LLM agent as something that "runs tools in a loop to achieve a goal."

## The Joy and Danger of YOLO Mode

Coding agents like Claude Code and Codex CLI default to requiring human approval for each command, which "dramatically reduces their effectiveness." The alternative — YOLO mode (automatic approval) — is described as "so dangerous" but essential for productive results.

Willison quotes Solomon Hykes: "An AI agent is an LLM wrecking its environment in a loop."

### Three key YOLO risks:
1. Malicious shell commands deleting or corrupting data
2. Exfiltration attacks stealing source code or secrets from environment variables
3. Using the machine as a proxy to attack other targets

### Mitigation strategies:
- Run in a secure sandbox (Docker, Apple's container tool)
- Use someone else's computer (GitHub Codespaces is his favorite — if things go wrong, it's "a Microsoft Azure machine somewhere that's burning CPU")
- Just take the risk (what most people do, he notes wryly)

He cites Anthropic's own documentation on "Safe YOLO mode" recommending `--dangerously-skip-permissions` in an internet-restricted container, linking to their .devcontainer reference implementation with a firewall script that limits access to a list of trusted hosts.

## Tool Selection for the Loop

Rather than leaning on MCP (Model Context Protocol), Willison prefers shell commands — coding agents are "really good at running shell commands." He creates an `AGENTS.md` file with example commands rather than extensive documentation. His example: providing one `shot-scraper` invocation as a template, which is "enough for the agent to guess how to swap out the URL and filename."

He notes that capable LLMs already know how to use tools like Playwright (Python) and ffmpeg, and since they operate in a loop, "they can usually recover from mistakes."

## Tightly Scoped Credentials

Two recommendations:
1. Use test/staging environments where damage is contained
2. Set tight budget limits on any credential that can spend money

Case study: He created a dedicated Fly.io organization with a $5 budget and a scoped API key to let Claude Code experiment with Dockerfiles for optimizing cold-start times — a project where he first recognized "designing an agentic loop" as an important skill.

## When to Use Agentic Loops

Best for problems with "clear success criteria" involving trial and error. He suggests the thought "ugh, I'm going to have to try a lot of variations here" is a strong signal to try this approach.

Examples given:
- Debugging — failing tests investigated by agents that can run them
- Performance optimization — benchmarking SQL queries, adding/dropping indexes in isolation
- Upgrading dependencies — with a solid test suite, agents can handle breaking changes
- Optimizing container sizes — iterating on Dockerfiles and base images

A common thread: automated tests massively amplify the value of coding agents. LLMs themselves are great for building those tests.

## Freshness of the Field

Willison emphasizes this is new — Claude Code was released in February 2025. He frames naming the concept as a way to "help us have productive conversations about it," noting there's "so much more to figure out."

## Links Referenced

- Claude Code: https://claude.com/product/claude-code
- Codex CLI: https://github.com/openai/codex
- Solomon Hykes quote: https://simonwillison.net/2025/Jun/5/wrecking-its-environment-in-a-loop/
- Apple container tool: https://github.com/apple/container
- GitHub Codespaces: https://github.com/features/codespaces
- Anthropic "Safe YOLO mode": https://www.anthropic.com/engineering/claude-code-best-practices#d-safe-yolo-mode
- Docker Dev Containers reference: https://github.com/anthropics/claude-code/tree/main/.devcontainer
- Firewall script: https://github.com/anthropics/claude-code/blob/5062ed93fc67f9322f807ecbf391ae4376cf8e83/.devcontainer/init-firewall.sh
- MCP: https://modelcontextprotocol.io/
- AGENTS.md: https://agents.md/
- shot-scraper: https://shot-scraper.datasette.io/
- Fly.io: https://fly.io/
