---
url: https://gist.github.com/9972636213519ecf3551adf4aa6d835f
title: "Essentials from a Real-World Microservices Journey — Sander Hoogendoorn (Craft 2025)"
author: Sander Hoogendoorn
date_fetched: 2026-09-13
date_published: 2025
topics:
  - software-engineering-craft
  - distributed-systems
---

A ytx gist holding both an auto-generated summary and the full transcript of Sander Hoogendoorn's Craft 2025 talk, delivered as CTO of iBood, a Dutch e-commerce company in six countries run by 13 developers across ~150–160 repos, deploying 40–50 times a day to production. His thesis: microservices aren't dead, but most teams adopt them for the wrong reasons — scalability being the canonical one ("most companies are just too small") — and the only justification that matters is destroying dependencies, because dependencies kill: they end in "technical death," where all engineering time is consumed keeping things up.

The talk walks Fowler's microservices definition as a table of contents. On decomposition: Gall's Law ("a complex system designed from scratch never works") backed by an insurance client's six failed two-year COBOL rewrites; DDD bounded contexts (Product means something different in replenishment than in ordering), aggregates turned into services with the root entity as the REST resource; cohesion redefined as code that changes together, not technical layers ("don't put all your repositories in one layer"). On architecture: every service shares one uniform four-layer micro-architecture (domain / repositories / use cases / resources), use cases double as the authorization unit, and everything sits on a single TypeScript stack so any of 13 people can open any of 160 repos and navigate it. The front end is split along the same domain seams into micro frontends that talk only to services, never databases.

On delivery: the "next bus in five minutes" metaphor — small releases make a bad release cheap; check-in to production in ~30 minutes through a pipeline defined as shared code (lint, unit tests, Sonar with 80% coverage measured on new code, containerize, acceptance, load tests); Feathers' unit-test rules; trunk-based development with no pull requests and no code reviews ("in his context" — he concedes regulated orgs with 30 teams may need them). On data: services own their data (shared databases just relocate the dependency graph), orders snapshot everything at purchase time, document databases by default because an aggregate persists in one statement. On ways of working: no Scrum, sprints, retrospectives, or product owners; event storming, pair/mob programming, self-organizing micro teams; Dee Hock's "simplify to amplify" illustrated by Amsterdam removing traffic lights.

The gist's own digest is unusually sharp about omissions: the monolith rebuttal is acknowledged then dodged (he even concedes cohesion works in a modular monolith — "you should, by the way"), observability is completely absent despite 150+ services, cross-service coordination and consistency beyond snapshots go unexplained, as do versioning, team scaling beyond 13 people, cost, and security beyond use-case scopes.
