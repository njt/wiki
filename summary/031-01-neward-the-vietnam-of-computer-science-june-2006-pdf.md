---
url: https://www.odbms.org/wp-content/uploads/2013/11/031.01-Neward-The-Vietnam-of-Computer-Science-June-2006.pdf
title: "The Vietnam of Computer Science"
author: Ted Neward
date: 2006-06-26
date_fetched: 2026-08-08
---

Ted Neward's classic essay argues that Object/Relational Mapping (ORM) is "the Vietnam of Computer Science" — a quagmire that starts with early successes, gets more complicated as time passes, and entraps its users in a commitment with no clear demarcation point, no clear win conditions, and no clear exit strategy.

Neward opens with a detailed historical analogy: the US involvement in Vietnam as a story of unclear goals, escalating commitment, and the slippery slope where further investment seemed like the only way to validate past sacrifices. He then maps this onto ORM: developers experience early wins mapping simple classes to tables, but as inheritance hierarchies, associations, and schema ownership conflicts emerge, the complexity compounds. Each additional feature (lazy loading, caching, query languages) introduces new failure modes without solving the fundamental problem.

The core technical diagnosis is the **Object-Relational Impedance Mismatch**: object systems are built on identity, state, behavior, and encapsulation, while relational systems are built on predicate logic and truth statements. These models are "too different to bridge silently." Neward catalogs six specific problems: the object-to-table mapping problem (especially inheritance), the schema-ownership conflict (DBAs vs. developers), the dual-schema problem (metadata in two places), entity identity issues (object identity vs. relational identity), the data retrieval mechanism concern (QBE → QBA → QBL — each with worse tradeoffs), and the partial-object problem / load-time paradox (you can't return "part" of an object).

He concludes with six possible responses: (1) abandon objects entirely, (2) abandon relational storage entirely (use OODBMS), (3) do the mapping manually, (4) accept ORM limitations and use raw SQL for the hard 20%, (5) integrate relational concepts into languages (presciently pointing to LINQ, Scala, F#), or (6) integrate relational concepts into frameworks. None is presented as the right answer — only that developers must "know when to cut bait and run" and avoid the slippery slope trap.

Written in 2006, the essay anticipated both the Hibernate/Entity Framework wars of the following decade and the eventual industry shift toward accepting that no ORM fully bridges the gap. Its Vietnam analogy — controversial then and now — was effective enough that the phrase entered the software engineering lexicon permanently.
