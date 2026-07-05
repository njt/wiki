---
url: https://www.railly.dev/blog/intent-layer/
date_fetched: 2026-07-05
backfilled: true
---

Today I’m releasing `/intent-layer`

`npx skills add crafter-station/skills --skill intent-layer -g`

Works with Claude Code, Codex, Cursor, Copilot, and  10+ more agents .

## The Problem

I’ve been using Claude Code daily for months. Same model, same prompts, completely different results depending on the repo.

On a large codebase I watched Claude:

- Burn 40k tokens exploring dead ends
- Find mocked tests, outdated docs, random utilities
- Miss the config file with the actual bug

Reasonable search. Wrong places. Bug still there.

## Why This Happens

Your best engineers don’t grep randomly. They have a mental map:

- What each folder owns
- What breaks if you touch it wrong
- Where the real logic lives

That map took years to build. Your agents don’t have it.

## The Solution: Context Engineering

Context engineering is designing the full information an agent needs to perform reliably:

- System prompts and instructions
- Structured inputs and outputs
- Tools and their definitions
- RAG and memory systems

Intent Layer solves the first piece: **system prompt infrastructure**.

## What Intent Layer Does

The skill helps you set up `AGENTS.md` files at folder boundaries. Simple markdown that gives agents the context they can’t get from code alone:

```
# Payment Service
## Purpose
Handles payment processing and settlement.
Does NOT own billing/invoicing (see billing-service).
## Contracts
- All processor calls go through src/clients/processor.ts
- Settlement config lives in ../platform-config/rules/
## Pitfalls
- src/legacy/ looks dead but handles pre-2023 accounts
- Test mode hits real sandbox. Charges appear then void.
```

Run `/intent-layer` on your project and it:

- Detects existing CLAUDE.md / AGENTS.md files
- Analyzes your codebase structure
- Suggests where to add context nodes
- Asks what patterns and pitfalls to document

Run it again later to audit existing nodes or find new candidates as your codebase grows.

## Results

Same bug, with AGENTS.md in place:

- 16k tokens loaded (not 40k)
- Went straight to the config file
- Found it first try

## Try It

`npx skills add crafter-station/skills --skill intent-layer -g`

## What’s Next

Intent Layer is the first of several context engineering skills I’m building. More coming soon.

 
## Credits

Built on  The Intent Layer  by Tyler Brandt. His  AI Adoption Roadmap  maps the stages most teams are stuck at. Start there for the full methodology.

Context engineering framework from  DAIR.AI  and  LangChain .

Also related to my earlier  AI-First Manifesto  where I proposed `LLMS.md` files. Intent Layer is the evolved version of that idea.


*Follow  @RaillyHugo  for more on context engineering.*
