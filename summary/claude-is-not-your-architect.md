---
url: https://www.hollandtech.net/claude-is-not-your-architect/
title: Claude Is Not Your Architect. Stop Letting It Pretend.
author: Charlie Holland
date_fetched: 2026-07-05
date_published: 2026-04-06
site: hollandtech.net
topics:
  - software-engineering-craft
---

# Claude Is Not Your Architect. Stop Letting It Pretend.

Charlie Holland, April 6, 2026. 7-minute read.

---

Holland opens by observing three organizations in the past month following the same pattern: someone asks an AI for architectural guidance, the AI responds with confident enthusiasm, and before long "Claude is the architect."

## The Attaboy Problem

AI agents are "pathologically agreeable" — trained to be helpful, which means agreeable. A real architect's primary value is saying "no," pushing back on complexity, and asking "why?" repeatedly until real requirements surface. An AI "will never do this."

## The Jenga Tower

The AI-produced architecture looks technically sound in isolation — recognizable patterns, passes what Holland calls "the squint test" — but it wasn't designed for any specific team's constraints: VPC lockdowns, legacy integrations, team inexperience with Kubernetes, compliance restrictions. It's designed for "the median of everything Claude has seen."

Real architecture requires contextual judgement: picking Postgres over DynamoDB because "your team knows Postgres" and you'd rather ship in two weeks than learn a new data model. "These decisions require judgement" and an understanding of actual organizational constraints.

## The Jira Ticket Pipeline

Holland's deeper concern is what happens after the AI designs the architecture: the same people ask it to produce epics, stories, and acceptance criteria. Engineers with years of contextual knowledge and experience are "no longer solving problems" but implementing Claude's design ticket by ticket. "The entity with the least context, no experience, and no accountability is making the architectural decisions."

## "But Someone Senior Signed Off"

Holland pushes back on the defense that a senior engineer reviewed the AI-generated proposal. A busy tech lead handed a coherent, well-terminologyed architectural document is unlikely to push back hard, especially when the counter-argument becomes "Claude spent twenty minutes on this and you want to throw it away?" The danger is that AI "short-circuits the discussion" — replacing messy, productive disagreement with deference to the tool.

## The Accountability Gap

"When it goes wrong, who carries the bag?" Not Claude. Claude doesn't get paged at 3am, doesn't sit through post-incident reviews, doesn't explain why the architecture couldn't handle load. The engineers do — the same ones who were implementing tickets from an entity that has "never operated a system in production."

## What to Do Instead

Holland uses Claude Code daily but says: "I tell it what to do, not the other way round." His prescriptions:

- **Engineers design. Agents implement.** Architecture comes from people who understand context, team, constraints, and politics. AI helps build it faster.
- **Challenge the attaboy.** Treat AI suggestions with the same skepticism you'd apply to "a confident junior engineer."
- **Protect the argument.** Messy disagreement between engineers is where good architecture comes from. Deference to AI replaces that valuable friction.
- **Keep humans accountable.** No human name on a decision means nobody owns it. "'Claude designed it' is not an architecture decision record."

## The Craft Still Matters

Holland notes the tool has evolved from whiteboard and opinion to AI agents, but "the craft hasn't changed" — understanding problems, knowing constraints, making trade-offs, defending simple solutions, saying no. That is architecture, and "no agent does it."
