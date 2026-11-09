---
url: https://lovable.dev/blog/how-we-built-lovable-and-why-we-keep-rebuilding-it
title: "How We Built Lovable, and Why We Keep Rebuilding It"
author: Anton Osika
date_fetched: 2026-11-09
date_published: 2026-11-04
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

Anton Osika's retrospective on building Lovable (from GPT Engineer to 70 million user-created projects), framed around a single principle: constrain the problem, control execution, and learn from the results.

Three architectural reversals stand out. First, autonomy was deliberately rolled back: an elaborate async subagent design by Mårten Wiman got stuck in rabbit holes with no predictable finish time, so the team deleted almost all agent code and replaced it with one predictable chain of tools per user request — a single response of file changes wrapped in XML-like tags (`<lov-write>`) parsed after the fact, with the human checking the live preview and steering. Second, the 2023 plan to power the product with fine-tuned models was replaced by "knowledge files": engineers read broken-app chat logs and wrote short instruction notes, retrieved into context per task — a skills-like pattern Osika credits as critical to Lovable's late-2024 take-off, "a scaling law" he half-jokes. Third, infrastructure for shortening the edit-check-correct cycle: persistent sandboxes keeping each project's dev server running, incremental TypeScript checks, and eventually checkpointing whole machines (memory, CPU state, disk) so an agent can fork a full working application environment and resume it days later.

The closing section names three open problems: putting the production lifecycle (testing, deploy, maintenance, trust — "one misunderstood request from a user can delete data") on autopilot; making all software types seamless; and building a better software stack outright — observability, forkable computer snapshots, new sync engines, durable execution, verifiable security.
