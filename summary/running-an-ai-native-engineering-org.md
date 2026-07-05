---
url: https://claude.com/blog/running-an-ai-native-engineering-org
title: "Running an AI-Native Engineering Org"
author: "Fiona Fung"
date_fetched: 2026-06-15
date_published: 2026-06-03
---

# Running an AI-Native Engineering Org

Fiona Fung, Director of Engineering for Claude Code and Claude Cowork, describes how the Claude Code team restructured their engineering processes around AI assistance. Written from direct experience leading the team, the post covers what broke, what replaced it, and how to tell if the new norms are sticking.

## Full Article Text

Fung opens by noting that "engineering bandwidth was the expensive part of building applications," with development processes built around that cost. She reflects on her early-2000s career on Visual Studio, when software shipped on CD-ROMs, then shifted to continuous online updates, and now undergoes another transformation around the time and people needed to write software.

On the Claude Code team, "writing code, writing tests, and refactoring rarely slows us down anymore." The bottlenecks moved to verification, code review, and security — not typing code.

### The processes that quietly stopped working

**Planning: Shift roadmaps to just in time**

The team initially wrote a six-month roadmap, but Claude Code made it obsolete by month three. Fung advocates "just-in-time (JIT) planning," likening it to JIT compiling. Planning shifted from design docs to discussions in PRs or prototypes, with less product review and more rapid prototyping with internal user feedback.

**Context gathering: Ask Claude, not the author**

Previously, answering questions meant finding the original code author. Since "all our PRs are assisted by Claude," the team now asks Claude directly and considers whether the question can be automated. Fung describes automating customer feedback summaries that she used to do manually.

**Code review: Trust but verify**

The team uses Code Review extensively. "Claude handles all the style and linting, PR feedback requests," catching bugs and adding tests. Human review is reserved for domain expertise — legal, trust boundaries, security-sensitive code, and product/design judgment. Fung notes the balance will shift as models improve.

**Team makeup: Blurring roles**

"PMs code a lot now" with Claude. Fung indexes on two hiring profiles: "creative builders with product sense" and "engineers with deep systems expertise." She deprioritizes raw throughput since "the models handle that."

The article includes a **Before/After table**:

| Area | Before | After |
|------|--------|-------|
| Planning | Six-month roadmaps | JIT planning: prototype, test internally, iterate |
| Context gathering | Ask the code author | Ask Claude first, then explore automation |
| Code review | Humans review everything | Claude handles style/bugs/tests; humans for domain expertise |
| Team makeup | Fixed roles | Blurred roles; hire for creative builders and systems expertise |

### How we rolled out new norms

Three core "non-negotiable must dos":

1. **"Relentlessly dogfood your product"** — every team member uses Claude Code and Cowork.
2. **"Keep the team flat as possible"** — managers start as ICs first; pods stay agile.
3. **"Don't hesitate to kill processes that no longer work"** — team members have explicit permission to question and remove obsolete processes.

Within these rules, pods have agency over their own rituals and workflows.

### How to know your new processes are sticking

Three metrics for engineering leaders:

- **Onboarding ramp time goes down** — engineers now ship real code within their first week.
- **PR cycle time goes down** — but may reveal bottlenecks in build systems or CI.
- **Claude-assisted commits going up** — Fung notes she hasn't "seen a non-Claude-assisted commit in the last four months."

She cautions: "don't confuse throughput with success."

### Getting started

Fung's closing advice: "pick your noisiest workflow" — the most expensive or dreaded one — and ask whether it still serves its purpose and can be automated. She recounts canceling a weekly review where everyone was on their laptops except during status reports. "Why are we having this meeting again?" sparked the realization it wasn't needed.

Her parting question: "what's one piece of your engineering workflow that you might consider automating or even dropping altogether?"

## Related posts

- "Beyond permission prompts: making Claude Code more secure and autonomous" (Oct 8, 2025)
- "How one Anthropic seller rebuilt his team's workflows with Claude Code" (Jun 5, 2026)
- "Lessons from building Claude Code: How we use skills" (Jun 3, 2026)
- "A harness for every task: dynamic workflows in Claude Code" (Jun 2, 2026)
