---
url: https://cloudedjudgement.substack.com/p/clouded-judgement-121225-long-live
title: "Clouded Judgement 12.12.25 - Long Live Systems of Record"
author: Jamin Ball
date_fetched: 2026-05-14
date_published: 2025-12-12
publication: Clouded Judgement (Substack)
topics:
  - databases-and-data
---

# Clouded Judgement 12.12.25 - Long Live Systems of Record

Jamin Ball argues that claims about systems of record dying — with agents or workflows replacing them — are misguided. He reframes "system of record" not as a product category but as the answer to "where does the truth live" in an enterprise.

## Core Argument

The ARR example: sales, finance, accounting, and legal each define ARR differently. When you instruct an agent to "Go calculate ARR by segment and send a deck to the board" — which canonical table should it use?

### Historical Evolution

Traditional systems of record (CRM, ERP, HRIS) each owned a domain. The warehouse/lakehouse era tried to centralize analytical truth but remained downstream — "the retrospective mirror, not the transactional front door."

### How Agents Change Things

Two shifts:
1. Agents are "inherently cross system" — dancing across CRM, CPQ, billing
2. Agents are "inherently action oriented" — changing state, not just running reports

This creates a "bull thesis on a company like Databricks" as a center of gravity for AI agents.

Agents force separation of the UX of work from the source of truth for work. The UX may be a chat window, but "something still has to say 'this is the canonical customer record.'"

### Warehouses as Substrate

Warehouses/lakehouses evolve from reporting systems into a "truth registry." The missing piece: these stacks were designed for humans, not agents. Agents need explicit rules, conflict resolution, and precedence encoded in data models.

Operational systems evolve into "state machines with APIs" optimized for programmatic access. "The human might still see that state in a web interface, but the primary consumer is an agent."

### AI-Native Apps

The most interesting ones sit next to existing systems rather than building new UIs. "Underneath the marketing, they are basically wrapping the messy reality of enterprise data in a cleaner contract."

### Valuation Angle

"An agent platform that becomes the place where metric definitions live...starts to look much more like a source of truth." The multiple follows "the stickiness of the truth, not the buzzword on the slide."

### Bottom Line

Agents are "raising the standards for what a good [system of record] looks like." Winners will build agentic experiences "on top of boring, rock solid sources of truth."

## SaaS Valuation Data

- Overall Median EV/NTM Revenue: 4.9x
- Top 5 Median: 22.7x
- High Growth (>22% NTM): 14.5x
- Mid Growth (15-22%): 7.5x
- Low Growth (<15%): 3.7x
- Median NTM growth: 12%
- Median Net Retention: 108%
- Median Gross Margin: 76%
- Median FCF Margin: 20%
- Median CAC Payback: 36 months
- Rule of 40 median: 8%

## Comments

- **La** questioned naivety around SOR complexity and asked about "CRM for AI native companies"
- **Arik Marmorstein** asked whether traditional UIs persist or chat interfaces suffice, comparing to a self-driving car's steering wheel

## Disclaimer

Ball discloses that views are his own and not necessarily those of Altimeter Capital Management, LP, and that the content is not investment advice.
