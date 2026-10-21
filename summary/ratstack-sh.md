---
url: https://ratstack.sh/
title: "Rat Stack"
author: Joel Hooks
date_fetched: 2026-10-21
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-coding-workflow
---

Joel Hooks' rat-stack is a TypeScript application-and-cloud reference stack, built on Effect (typed dependencies, errors, concurrency) and Alchemy (cloud resources declared in TypeScript), shipped on Cloudflare. The site is explicitly agent-first: "Every page is Markdown by default", with an llms.txt agent guide, an MCP server, and session-scoped skills — you paste a connection prompt into your coding agent and it learns the stack's patterns before writing code.

The design thesis is trust through the compiler: "Make the easy path the right path. The codebase and the compiler stop mistakes that rules and style guides can only ask about." The "fence" — compiler checks, lint rules, commit hooks — rejects prohibited code outright, so agents can work longer without human help. Remove a cloud resource and the code that uses it stops compiling.

Structurally, it's a package-per-job ("cartridge") architecture: contract → capability → projections, where one Effect handler is exposed as CLI, HTTP+OpenAPI, MCP tools, browser RPC, and sandboxed code mode. Hexagonal architecture keeps provider-specific adapters out of core; removal checklists let you cut any package and leave unrelated ones working. The stack is CAFE: Cloudflare, Alchemy, Foldkit, Effect.
