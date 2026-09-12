# Engineering Atlas Articles

The article corpus behind the [[Software Engineering Practice Atlas]]: 5,172 crawled engineering reports — blog posts, incident reports, and talks from sources like Hacker News, The Pragmatic Engineer, Cloudflare, and ByteByteGo — each classified along three axes: **outcome** (Win, Migration, Lesson, Outage, Postmortem), **phase** (Process → Maintenance), and **quality attribute** (Correctness → Compliance). It is the evidence layer beneath the atlas's synthesized practice cards, exposed as a filterable feed whose real product is not the articles but the taxonomy that organizes them.

---

## Key Quotes

The classification is the content. The feed's filter bar is a three-axis taxonomy for engineering writing, and its counts are the most honest summary of what engineers actually write down:

> "Outcome: All outcomes 5172 — Win 1620, Migration 130, Lesson 2734, Outage 566, Postmortem 122"

> "Phase: All phases 5172 — Process 1049, Requirements 906, Design 1700, Development 520, Testing 575, Releasing 249, Operation 481, Maintenance 163"

> "Quality attribute: All attributes 5172 — Correctness 2981, Reliability 1649, Performance 984, Scalability 763, Security 550, Maintainability 1286, Operability 745, Usability 819, Cost 1204, Compliance 561"

The entry summaries show the feed's texture — each is a human-written piece, machine-summarized and machine-classified:

> "Schema changes look trivial, such as renaming a column or adding a field to an event, but they are often some of the riskiest changes in a distributed system." *(ByteByteGo, "Schema Evolution: Changing the Contract Without Breaking What Runs")*

> "The article argues that radians are an unnecessary intermediate representation in most code, and that angles should be parameterized as turns (full circle = 1) instead." *(Hacker News, "Turns are Better than Radians")*

Every entry closes with a "Relates to" list that wires the article to concept cards — "Prompt Engineering", "Least-Privilege Access", "Expand/Contract Migrations", "Do Not Tamper With a Stable Process". That wiring, not the article itself, is the site's actual value.

---

## Key Themes

#tool #concept #taxonomy #curation #engineering-writing

### Outcome is a partition; phase and attribute are facets

The three axes are not symmetric, and the numbers prove it. Outcome counts sum to exactly 5,172 — every report gets exactly one of Win/Migration/Lesson/Outage/Postmortem. Phase counts sum to 5,643 (~1.1 per report) and quality attributes to 11,542 (~2.2 per report). So **outcome is a partition** — the spine of the taxonomy — while phase and attribute are facets a report can bear several of. The site puts outcome first in the filter bar because it knows this: the primary organizing principle of engineering knowledge here is *what happened*, not *what it's about*.

### The evidence layer beneath the synthesis

The Practice Atlas's cards link to "crawled engineering blog articles." This is that corpus, exposed as its own index. The site is therefore two layers: synthesized practice cards (AI-generated, admittedly unedited) on top, and the raw article feed (human-written, machine-classified) underneath. That separation matters — it's the difference between a claim and its evidence. When a card is plausible-but-wrong, the articles are where you go to check.

### What the distribution reveals

The counts are a census of what engineers *choose to write*:

- **Lessons dominate.** 2,734 of 5,172 reports (53%) are "Lesson". More than half of engineering's written record is "here's what I learned."
- **Wins are second** (1,620, 31%). Outages and postmortems together are 688 (13%) — failure reporting is a meaningful but minority genre.
- **Migrations are nearly absent** (130, 2.5%). Yet the feed's own Schema Evolution entry calls schema changes "some of the riskiest changes in a distributed system." The riskiest, most consequential work is the least written about.
- **Correctness is the modal attribute** (2,981 — tagged on 58% of all reports), ahead of reliability (1,649) and maintainability (1,286). Security sits near the bottom (550) despite being the attribute every org claims to care about. Cost (1,204) is surprisingly high, an artifact of the AI-era cost panic.

---

## Critical Analysis

**The taxonomy is the product.** The articles themselves are a commodity — any reader can get them from HN or an RSS reader. What the atlas adds is the three-axis classification and, more importantly, the "Relates to" wiring that connects each article to concept cards. That turns a pile of links into a navigable knowledge graph. The feed is the showcase; the graph is the product.

**The outcome partition is a bold, machine-made claim.** Calling a report a "Win" or a "Lesson" is an act of interpretation, not a neutral fact — and it's machine-assigned, consistent with the site being AI-generated. A postmortem that teaches a lesson is also a Lesson; a migration that went well is also a Win. The single-outcome partition flattens this nuance. But the flattening is also what makes the feed browsable — you can't facet on "it's complicated."

**The counts measure the written record, not importance.** Security is not one-fifth as important as correctness because it has one-fifth the articles; migrations are not trivial because they're rare. The atlas classifies what engineers wrote down, and the written record has its own biases — the same point [[Notes and Queries — Victorian Crowdsourced Knowledge]] makes about periodicals: the format selects for what gets recorded. A "Lesson" is a satisfying thing to publish; a half-failed migration is not.

**The taxonomy-first/lazy-first split.** This is the mirror image of [[Immaculate Knowledge Graph]], where Harper Reed refuses to design a taxonomy upfront — "just make it work for you." The atlas imposes a designed three-axis scheme on all 5,172 reports. The upfront taxonomy scales (thousands of items need a fixed schema) but bakes in its categories' blind spots; the lazy approach doesn't scale but never forces material into boxes that don't fit.

**A structural echo of this wiki.** The atlas's two-layer design (verbatim articles beneath synthesized cards) is the same shape as this wiki's three-tier layout (raw → summary → topic). The atlas adds one thing this wiki mostly lacks: a formal facet scheme that makes the corpus browsable by outcome/phase/attribute, not just by search. That's the experiment worth watching — whether a fixed taxonomy ages better than free [[wikilink]]-driven association.

---

## Cross-References

- [[Software Engineering Practice Atlas]] — the synthesis layer this corpus feeds. The cards are the claims; these articles are the evidence.
- [[Notes and Queries — Victorian Crowdsourced Knowledge]] — the same format-shapes-knowledge dynamic, two centuries apart.
- [[Immaculate Knowledge Graph]] — the lazy-first opposite of the atlas's taxonomy-first approach.
- [[A Pattern Language (Christopher Alexander)]] — the pattern-catalog lineage the atlas descends from.
- [[Software Engineering Craft]] — the hub this external reference overlaps with.

---

*Sources: [[raw/articles]], [[summary/articles]]*
*Last updated: 2026-08-21*
