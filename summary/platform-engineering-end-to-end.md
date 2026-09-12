---
url: https://www.lucavall.in/blog/platform-engineering-end-to-end
title: "Platform Engineering End-to-End"
author: Luca Cavallin
date_fetched: 2026-05-18
date_published: 2026-05-06
topics:
  - software-engineering-craft
---

# Platform Engineering End-to-End

Luca Cavallin's comprehensive field guide to platform engineering, drawn from Fournier and Nowland's book and his own GCP experience. Covers definitions, team structure, product management, operations, migrations, stakeholder politics, and a priority order for starting from zero.

## Core Definition

"a platform team builds and operates an internal product whose users are other engineers." Simple but not easy.

## Why Platform Engineering Exists

Cloud and OSS created a landscape of endless primitives — multiple queue options, object stores, databases, CI runners, service meshes. Teams pick different stacks, and within a year infrastructure becomes "a swamp of glue code where every service has its own deploy pipeline."

Four core functions:
1. Limits the primitives developers see — curated, opinionated pathways
2. Reduces per-application glue by absorbing repetitive plumbing into shared services
3. Centralizes the cost of migrations — platform team handles underlying changes once
4. Lets developers operate what they build without forcing deep infrastructure expertise

Key distinction from DevOps: "DevOps said 'developers, take ownership of operations'. Platform engineering says 'fine, but we will give you good tools to do that.'"

## The DORA 2025 Data

90% of organizations have adopted at least one internal platform. Platform quality now predicts whether AI tooling produces value or chaos. "A bad platform makes AI tools amplify chaos. A good platform makes them amplify throughput."

## The Five Pillars

1. **Curated Product Approach** — "Saying no is part of the job." The response to a request for Kafka over Pub/Sub should include supported options, rationale, and an off-ramp for genuinely exceptional cases.

2. **Software-Based Abstractions** — The platform's interface is APIs, CLIs, and SDKs — not wikis or Slack pings. Score project (CNCF): a small declarative workload spec provisions databases, topics, service accounts, and deployments without the developer caring about Cloud SQL, Pub/Sub, or Cloud Run underneath. "The developer does not care that under the hood it is Cloud SQL, Pub/Sub, and Cloud Run. That is the point."

3. **OSS Customizations and Metadata Registries** — Run open-source tools (Argo CD, Backstage) with org-specific plugins. Maintain a metadata registry / service catalog as single source of truth for service ownership and dependencies. Backstage: 270+ organizations in production.

4. **Serving a Broad Base** — "The platform exists to serve the median developer doing the median task, well." Building only for elite users causes the long tail to work around the platform.

5. **Operating as Foundations** — "If your platform is down, the company is down." Requires 24/7 on-call, real SLOs, real change management.

## When to Start

Don't form a platform team too early. At ~10 engineers, cooperation suffices. A premature 1-2 person team becomes a ticket queue. Form when cooperation visibly breaks, "usually somewhere past 50 engineers."

Transforming infra orgs: the hardest shift is cultural. Infra people are gatekeepers of "no"; platform people must become providers of "here is the easy yes."

## Team Composition

Four roles: software engineers (APIs, SDKs, portals), systems engineers (Kubernetes, Linux, networking), reliability engineers (SLOs, on-call, observability), systems specialists (domain expertise in databases, security, networking).

"Hire for customer empathy. I cannot stress this enough." A platform engineer who cannot sit with a frustrated app developer is in the wrong job.

Managers: must have operated platforms (not just built them), shipped long-running multi-quarter projects, and be obsessive about details — "the 1% of skipped cases will consume 80% of support time."

## Platform as a Product

Internal customers are captive — cannot churn easily, have strong opinions and weak product instincts. Empathy wins: sit with them, watch them work, count context-switches per change. "That is your real backlog."

Roadmap: Vision (multi-year) → Strategy (annual bets) → Goals/metrics (quarterly to annual) → Milestones (quarterly deliverables).

Common failures: underestimating migration cost ("always 2-3x what you think"), overestimating change budget, adding features when stability is the problem, too many PMs relative to engineers.

## Operations

24/7 on-call is non-negotiable: "The team that builds the deploy system is also the team that gets paged when it breaks at 2 a.m." This is a feedback loop, not punishment.

Four stages of support maturity: formalize → separate non-critical from on-call → hire dedicated support → build engineering support org.

The DORA 2025 finding: "the platform capability most correlated with positive user experience is clear feedback on task outcomes."

## Migrations

v2 projects "almost always wrong." Instead: rearchitect inside the existing platform, maintain compatibility, tranche-based migration. Security must be architectural: "You cannot bolt security onto a platform after it is built."

Migration antipatterns: asking every team to do it themselves with a clipboard, mandating without on-ramps, underestimating the long tail.

"Mandates work once or twice, then they become noise." Make the new path so much better the old path withers.

## Stakeholder Management

Power-interest grid for stakeholder mapping. Senior leaders need milestone/risk updates, not gRPC retry debates. Saying no: be clear about business impact — "if we add this feature, our migration slips by a quarter, which costs the company $X."

Budget season: group by team and capability, not by person. "If you don't [come with strong opinions about cuts], finance will pick for you and they will pick wrong."

## What Success Looks Like

- **Aligned platforms**: multiple teams pulling same direction on purpose, strategy, plans
- **Trusted platforms**: built slowly, lost in a single bad migration
- **Platforms that manage complexity**: accidental complexity (from sloppy coordination, shadow platforms) is not unavoidable
- **Loved platforms**: "If your platform is loved, you can ask for a budget and people will fight for you. If it is tolerated, you are one bad incident away from being replaced."

## Priority Order from Zero

1. Decide what you support and don't — write it down, defend it
2. Invest in software abstractions, not wiki pages (Score, Crossplane, custom SDK)
3. Stand up a metadata registry (Backstage or similar)
4. Build for the median team, not the loudest one
5. Treat operations as a first-class feature (SLOs, on-call, support tiers)
6. Hire for empathy as much as for systems chops
7. Communicate ruthlessly (biweekly wins/challenges, transparent roadmaps, honest stakeholder management)
8. Cut what you don't need — sunset, consolidate, say no

## Closing

"If you want to build something that real engineers depend on every day, that compounds in value over years, and that you can defend to a CFO with actual numbers, this is one of the most leveraged places to spend a career."

Cavallin credits Fournier and Nowland's book as the primary source; "the mistakes are mine."
