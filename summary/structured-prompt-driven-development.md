---
url: https://martinfowler.com/articles/structured-prompt-driven
title: Structured-Prompt-Driven Development (SPDD)
author: Wei Zhang and Jessie Jie Xia
date_fetched: 2026-05-18
date_published: 2026-04-28
topics:
  - agent-coding-workflow
---

# Structured-Prompt-Driven Development (SPDD)

Published on Martin Fowler's site. Authors: Wei Zhang and Jessie Jie Xia. Tag: generative AI.

## Core Argument

SPDD treats prompts as first-class delivery artifacts — version-controlled, reviewed, and kept in sync with code — to make AI-generated changes governable, reviewable, and reusable at team scale. Individual developer speed from AI assistants doesn't automatically translate to system-level throughput: "buying a Ferrari and driving it on muddy roads: the engine is powerful, but your arrival time is determined by road conditions and traffic."

Closing thesis: "In the AI era, software development isn't a contest of model IQ. It's a contest of engineer cognitive bandwidth."

## The REASONS Canvas (7-part prompt structure)

- **R** — Requirements (problem + Definition of Done)
- **E** — Entities (domain concepts, relationships)
- **A** — Approach (strategy to meet requirements)
- **S** — Structure (where the change fits in the system)
- **O** — Operations (concrete, testable implementation steps)
- **N** — Norms (cross-cutting engineering standards)
- **S** — Safeguards (non-negotiable boundaries)

## The SPDD Workflow (6 steps)

1. Create initial requirements (optionally using `/spdd-story`)
2. Clarify analysis (human reviews core logic, scope, DoD)
3. Generate analysis context (`/spdd-analysis`)
4. Generate structured prompt (`/spdd-reasons-canvas`)
5. Generate code (`/spdd-generate`) + verify + review/adjust
6. Generate unit tests

**Golden rule:** "When reality diverges, fix the prompt first — then update the code."

## Key Commands (openspdd CLI)

- `/spdd-story` — Optional. Break requirements into INVEST-compliant user stories
- `/spdd-analysis` — Core. Extract domain keywords, scan code, produce strategic analysis
- `/spdd-reasons-canvas` — Core. Generate full REASONS Canvas
- `/spdd-generate` — Core. Generate code from Canvas, task by task
- `/spdd-api-test` — Optional. Generate cURL-based API test scripts
- `/spdd-prompt-update` — Core. Incrementally update Canvas when requirements change
- `/spdd-sync` — Core. Sync code-side changes back into Canvas

## Three Core Skills for Developers

1. **Abstraction first** — "design before you generate"
2. **Alignment** — "lock intent before you write code"
3. **Iterative Review** — "turn output into a controlled loop"

## Fitness Assessment

- **High fit (★★★★★):** Scaled standardized delivery, high-compliance environments
- **Medium fit (★★★★☆):** Team collaboration/auditability, cross-cutting consistency work
- **Low fit (★★☆☆☆):** Firefighting hotfixes, exploratory spikes, one-off scripts
- **Poor fit (★☆☆☆☆):** "Context black holes" (unclear domain), pure creative/visual work

## Key Quotes

- "The real question isn't 'How do we generate more code?' It's how do we make AI-generated changes governable, reviewable, and reusable."
- "In engineering, if you don't know what you are doing, you shouldn't be doing it." — Richard W. Hamming
- "The loop is closed by the workflow and the artifact, not by an autonomous learning mechanism."
- SPDD is "spec-anchored" (per Birgitta Böckeler's categorization) — distinct from spec-first or spec-only approaches.

## Example: Billing Engine Enhancement

Token-billing system enhanced to support multi-plan, model-aware pricing. GitHub: gszhangwei/token-billing. ~99% intent alignment, complete engineering transparency, synchronized prompt assets.

Two types of code review adjustments:
- **Logic corrections** → update prompt first (`/spdd-prompt-update`), then regenerate
- **Refactoring** → change code first, then sync back (`/spdd-sync`)

## Notable

- Authors acknowledge Martin Fowler's narrative and diagram contributions
- Article itself was shaped with LLM assistance (Claude, GPT, Gemini)
- `openspdd` is open source on GitHub
- Article includes lengthy Q&A on: developer variance, model agnosticism, scaling, hotfix handling, roadmap
