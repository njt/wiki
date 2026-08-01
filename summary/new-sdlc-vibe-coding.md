---
url: https://drive.google.com/file/d/1IR7CddF_2FyQo_PdfBNTaEA50EGiVt2r/view
title: "The New SDLC with Vibe Coding: From ad-hoc prompting to Agentic Engineering"
author: Addy Osmani, Shubham Saboo, and Sokratis Kartakis
date_fetched: 2026-07-18
date_published: 2026-05
---

A paper by Addy Osmani, Shubham Saboo, and Sokratis Kartakis that maps the emerging landscape of AI-assisted software development. It defines a spectrum from casual "vibe coding" — prompting an AI and accepting whatever comes back with minimal verification — to disciplined "agentic engineering," where AI acts as an implementation engine inside structured systems of specifications, tests, guardrails, and human architectural oversight.

The central insight is that the quality of AI-generated code depends less on clever prompts and more on context engineering: the deliberate management of what an agent knows upfront (static context) versus what it fetches on demand (dynamic context). Agent Skills — structured, portable packages of procedural knowledge loaded only when relevant — are presented as the most powerful pattern for dynamic context. The paper also introduces the harness concept: the scaffolding around a model (instructions, tools, sandboxes, hooks, observability) that turns a raw model into a working agent. Harness changes alone can swing agent performance dramatically on benchmarks, with no model change.

The authors work through every SDLC phase, showing how AI is reshaping requirements, architecture, implementation, testing, code review, and maintenance — not by replacing human developers but by shifting their role from writing code to reviewing, guiding, and verifying it. They name the "80% problem" (AI does the first 80% fast, the last 20% needs deep context), contrast the factory model of development, and lay out concrete starting points for individual developers, engineering leaders, and organizations. The economics section is particularly sharp: vibe coding looks cheap upfront but carries compounding operational costs, while agentic engineering requires higher initial investment with dramatically lower marginal cost per shipped feature.
