---
url: https://gist.github.com/178d71dd271ddba3a22bd71c5ce45cba
title: "Evolutionary Architecture: The What. The Why. The How."
author: Maciej "MJ" Jedrzejewski
date_fetched: 2026-09-13
date_published: unknown
topics:
  - software-engineering-craft
---

A ytx gist transcription of Maciej "MJ" Jedrzejewski's architecture talk at the Craft conference in Budapest (year undated in the transcript), consisting of the transcriber's own four-part digest — key points, pithy quotes, toolkit, unanswered questions — followed by the full transcript and Q&A.

MJ's diagnosis is the **Project Paradox**: architecture forces its biggest technical decisions at the beginning, exactly when domain knowledge is minimal, and by the time you actually understand the domain there are no big decisions left to make. His one-sentence takeaway: "Always choose an architecture based on your current needs and based on your current context. Not a wishful thinking." No building for imaginary million-user futures — but also "don't close the door": keep technical options (like swapping the relational database later) open as long as possible.

The prescription is a four-chapter lifecycle — **Simplicity → Maintainability → Growth → Complexity** — with growth explicitly optional ("not all products will come to that point") and transitions triggered by observed architectural drivers (performance, complexity, maintainability), never vanity metrics. He walks it through a healthcare case study (private medical clinics): requirements grouped into highly cohesive areas (appointment scheduling, patient treatment, drug prescription, medical records, invoicing), mirrored as folders, each with its own database schema, tested by the "Formula One driver shouldn't prepare your sandwich" cohesion heuristic.

The operational core is a pair of ladders and a deferral discipline. Scaling: fix indexes and queries → read replicas → cache last ("Yesterday I had one problem and I added cache to solve it. Now I have two problems."). Messaging: in-memory queue → Postgres-as-queue with inbox/outbox → message broker only once deployment units actually split — "leverage what you have." A single deployment unit survives all the way to 10,000 patients and three teams; extract a microservice only when teams "step on each other's toes," a module needs different security, or one module dominates (patient treatment: 40% of traffic). Chapter four fights complexity in the code, not the topology: event storming reveals the anemic prescription entity is mutated by several outside services, and the aggregate becomes the "guardian of consistency." DDD verdict: strategic yes from day one (subdomains → bounded contexts → context map), tactical DDD not at the start — "If you skip the strategic part, then you are hacked up" — and bounded contexts are not permanent ("If you put trucks into cars, then you will have a mess").

The gist's own digest is unusually honest about the holes: no trigger metrics (how much performance loss justifies extraction?), no migration path between chapters (the mid-flight data-migration question was punted as "opening a Pandora box"), eventual consistency hand-waved in the riskiest possible domain, no observability story despite triggers being the whole mechanism, and the tension between "not wishful thinking" and "don't close the door" — optionality is itself speculative structure — left unresolved.
