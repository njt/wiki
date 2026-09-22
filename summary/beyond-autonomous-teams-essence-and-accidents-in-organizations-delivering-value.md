---
url: https://gist.github.com/fd17a41b0eb5af024b552a9cec6f0d8e
title: "Beyond Autonomous Teams — Essence and Accidents in Organizations Delivering Value"
author: Simon Rohrer
date_fetched: 2026-09-13
date_published: 2026
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

Simon Rohrer's Craft 2026 talk argues that "autonomy" is the wrong word and the wrong goal for teams. Etymologically it means self-law (autos + nomos), subject to one's own laws — which no team in a complex organization actually wants or should have. It survives as a container concept (like "sustainability" or "quality") that people fill with whatever they please; the German socio-technical tradition is more humble (teilautonom, partially autonomous) and Dutch/French/Hungarian say self-steering. His replacement: **agency** — capacity to act within clear boundaries — paired with **coherence**.

"Product" gets the same treatment. Scrum descends from the New Product Development Game — brand-new standalone products — but in a complicated organization what one team delivers is rarely a product; his Barclays group tried "service" and gave up: "it's a banana." The proposal is **value centers** at every level of the hierarchy, borrowed from cost centers but flipped: every team exists because it delivers value, not because it incurs cost.

The talk's structural claim is the **shape of value**: a cloud hyperscaler sells tiny atoms (storage, compute, DNS) that can be owned by two-pizza teams, while a trading platform has one product sold by 80+ teams around a large shared kernel (orders, prices, risk, portfolios). Essential vs. accidental dependencies: the accidental can be decoupled with contracts and APIs, but the essential coupling *is* your value — you can't DDD or Team Topologies it away. Extending Conway (via Sanchez & Mahoney 1996), value designs organizations, fractally, at every level.

On purpose and governance: Stafford Beer's "the purpose of the system is what it does" (a hospital A&E losing patients sicker has the purpose of making people sick); strategy negotiated both up and down (Roger Martin's call-center worker); governing constraints for ordered domains, enabling constraints (platforms) for disordered ones — with Novo Nordisk's security incident as the cautionary tale of treating security governance as enabling when it had to be governing. He modernizes Beer's Viable Systems Model into five questions every value center must answer at every level (value, coordination, fit, environment, identity), balanced by the Better Value, Sooner, Safer, Happier scorecard.

The AI coda is short but pointed: Meta measures agency as hours without human intervention — "I don't think that's what agency is." The same agency-within-boundaries frame applies to agents (the boundary is the harness), five open agent tabs raise the value-center question (what value is each delivering? how do you make them coherent?), and he names **context engineering** as "the most important thing for us to be concentrating on in the world of AI agents" — sharing skills and context across individual, team, department, and organization (citing Patrick Debois) — while admitting the details need another forty minutes.
