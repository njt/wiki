---
url: https://processflow.tech/
title: "Process Flow — Simple Service Orchestration"
author: unknown
date_fetched: 2026-06-09
date_published: unknown
topics:
  - developer-tools
---

Process Flow is a SaaS service orchestration platform for API-first teams. It uses a choreography pattern: each workflow stage is an HTTP endpoint that returns JSON specifying the next stage's URL, execution time, and updated state. There is no central workflow definition — workflows emerge as chains of stages, each designating its successor.

## How it works

1. **Create**: Define a workflow with a first stage URL and execution time
2. **Execute**: Process Flow calls the stage, records the run, and builds the next stage from the JSON response
3. **Continue**: Each stage designates its successor, forming a chain

Each stage receives the full workflow state, then returns updated state plus the next service location and scheduled execution time.

## Key features

- **Dynamic workflow design**: No restrictive contracts; each stage defines the following stage URL, execution time, and state
- **Complete visibility**: Examine any stage's state and execution history. Pause, reschedule, cancel, or edit stages from the web UI
- **Branching and re-runs**: Branch workflows from any stage, re-run stages at any time
- **Workflow control**: Cancel, pause, or reschedule stages. Modify workflow state
- **Human-in-the-loop**: Web interface for inspection and intervention
- **API-first**: All operations available via API

## Pricing

All tiers priced in GBP (£), suggesting UK-based:

| Tier | Executions/month | Price/month |
|------|-----------------|-------------|
| Developer | 500 | Free |
| Hobby | 2,500 | £30 |
| Starter | 5,000 | £50 |
| Growth | 10,000 | £100 |
| Business | 25,000 | £250 |
| Scale | 50,000 | £500 |

Overage: £10 per 1,000 additional stages.

## Tech stack

- **Frontend**: SvelteKit (SSR), Material Dashboard v3.1.0
- **Auth**: Google Sign-In
- **Icons**: Nucleo, Font Awesome, Material Symbols
- **Charts**: Chart.js
- **Drag-and-drop**: dragula, jkanban
- **Additional**: GitHub buttons, Popper.js, Bootstrap JS, Perfect Scrollbar

The site serves SSR HTML (readable without JS) and hydrates into a SvelteKit SPA. The page blocks automated fetchers (WebFetch returns 403) but serves full content to browsers.
