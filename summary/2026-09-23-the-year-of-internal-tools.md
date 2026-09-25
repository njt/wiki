---
url: https://www.geocod.io/code-and-coordinates/2026-09-23-the-year-of-internal-tools
title: "The year of internal tools"
author: Geocodio (Francesca or Jeff, unattributed on the blog)
date_fetched: 2026-09-25
date_published: 2026-09-23
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

Geocodio's engineering blog declares 2026 "the year of internal tools." Frontier models changed the economics: what used to be bash scripts and artisan commands is now full-fledged, mobile-friendly internal apps (Atlas for customer support, Bullpen for sprint planning, Yak the open-source papercut agent, Treehouse for per-branch worktrees). Crucially, the authors claim AI solved the *other half* too — maintenance, historically the thing that killed ambitious internal tooling.

Their system: weeks of planning and spec-writing, engineering spikes, repeated Grill Me sessions (Matt Pocock's adversarial skill) to stress the plan, UI mockup iteration before building, and shared infrastructure — a console-ui package with shared Tailwind tokens and React components so apps don't each grow their own. "The planning and the spec take weeks. The implementation takes hours or days." Every tool ships a full documentation subsite, largely AI-drafted, kept current by a CLAUDE.md rule requiring doc updates in the same change, backed by a DocsManifest as source of truth.

They are deliberate about limits: three questions gate build-vs-buy (domain experience, blast radius of downtime, does value come from integration with their own data). No staging environments for internal tools — production deploys with local QA, tests, and CI carrying the weight; an ETL outage showed what that decision costs when it goes wrong. Tools live inside the private network behind a VPN. The closing advice: start small, close the gap you already work around by hand.
