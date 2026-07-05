# Open Source AI Gap Map (Willison)

Simon Willison's link-blog take on Current AI's Gap Map — an interactive index of the open-source AI ecosystem backed by MIT-licensed YAML data. Simon's lens is characteristic: he's less interested in the pretty visualization than in the underlying dataset, and he immediately puts it to work with his own tools (Datasette Lite) to demonstrate what the data makes possible.

---

## What Current AI built

Current AI is "a global partnership building a public option for AI," founded as a non-profit at the AI Action Summit in Paris (February 2025) with $400m already committed. Their Gap Map v0.1 catalogs the open-source AI ecosystem:

> The Gap Map v0.1 details 421 products in depth: 266 software tools and libraries, 85 models, 50 datasets, and 20 hardware projects, produced by 228 organizations. These products are organized into 14 categories across 3 layers of the stack (model components, product / UX, and infrastructure).

The remaining 24,400 artifacts sit in the uncategorized long tail — discovered but unscored until an analyst gets to them. This honesty about coverage (421 scored out of ~25K known, roughly 1.7%) is itself a design choice worth noting.

## Simon's angle: the data, not the map

Simon's characteristic move is to look past the surface and find the reusable part:

> The map itself is interesting to explore, but I'm more excited about the underlying data — released under an MIT license in the currentai-org/os-ai-map GitHub account: 1,184 YAML files plus the notebooks, schemas and other scripts used to help gather them.

He then does what Simon always does: wires it up to his own tools. He loads a CSV of 16,185 GitHub repos the project is tracking into Datasette Lite, making the raw data instantly queryable in the browser with no server. This is the Willison pattern: find structured data, put it in SQLite, give people a way to ask questions.

---

## Key Themes

### #concept — Data as the durable artifact

The map is a snapshot; the dataset is infrastructure. Simon's excitement about the MIT-licensed YAML over the interactive visualization reflects a conviction that runs through all his work: the raw data outlives the presentation layer. If you have structured, queryable data with clear provenance, you can build any visualization on top of it. The inverse is not true.

### #pattern — Datasette as universal data browser

Simon's reflex — "here are 16,185 GitHub repos as a CSV loaded into Datasette Lite" — is a pattern he's refined over years. Whenever he encounters structured open data, he routes it through SQLite. Datasette Lite (WebAssembly SQLite in the browser) means anyone can query it without installing anything. The tool choice is the argument: data deserves a query interface, not just a pretty map.

### #person — Simon Willison as curation infrastructure

This post is a link-blog entry — short, pointing outward, no original reporting. But Simon's curation IS the value. He reads the landscape, identifies what matters, extracts the reusable part, and shows you how to use it. Fowler and Beck name-checked him as the counterweight to their own skepticism about AI autocomplete — the person whose honest, "I don't know when I don't know" coverage kept them from dismissing the field. This post is that pattern in miniature.

### #tool — Current AI as public-option infrastructure

Current AI's framing as "a public option for AI" is significant. In a landscape where open-source AI data is scattered across Hugging Face, GitHub, and arXiv, a curated, scored, MIT-licensed index with citation trails is itself infrastructure. It's the kind of thing that should be public goods — like OpenStreetMap for the AI stack.

---

## Critical Analysis

**The gap between the map and the data is the whole story.** The interactive map is fine — 14 categories, 3 layers, color-coded maturity stages. But Simon's instinct to ignore it and go straight to the YAML files is correct. The map answers "how mature is this category?" The data answers "what actually exists, and can I query it?" The second question is vastly more useful. Current AI should lean harder into being a data provider and treat the map as a demo, not the product.

**Datasette Lite as a stress test.** Simon's demo — loading 16K rows into browser-based SQLite — is also an implicit critique of the project's own interface. If a single journalist can make your data more queryable in five minutes than your own interactive visualization does, your UX strategy needs work. The map is a brochure; Simon turned it into a database. That's the gap that matters.

**The curation bottleneck is the real story.** 421 scored products out of 25K known is a 1.7% coverage rate. This isn't a failure — it's honest. But it means the map's maturity judgments are operating on a tiny sample. The long tail might contain products that would change stage assignments. The methodology acknowledges this, but the interactive visualization doesn't communicate it clearly. Simon's Datasette demo, by making the uncategorized mass visible, does.

**Willison's curation compounds.** This is at least the tenth Simon Willison post in this wiki. Each one is short, but the cumulative effect is a map of the AI landscape that's more useful than any single deep-dive. He's not a researcher or a builder of the things he covers — he's a reader who shares what he finds, and that role turns out to be infrastructure. The wiki's own [[2025 in LLMs]] page, [[Understand to Participate]], [[HTML Table Extractor]], and [[Designing Agentic Loops]] all trace back to his blog. He's the person who tells you what's worth paying attention to, and in a field moving this fast, that's a superpower.

---

## Related Pages

- [[Open Source AI Map]] — Deep technical analysis of the same currentai-org/os-ai-map project: architecture, methodology, scoring system, and design decisions
- [[2025 in LLMs]] — Simon Willison's annual landscape survey; the Gap Map is a natural complement
- [[HTML Table Extractor]] — Another Willison post where structured data + browser tools is the pattern
- [[How Far Behind Are Open Models]] — Quantified open-vs-closed gap; Current AI's openness axis addresses the same question from a different angle
- [[AI Value Chain]] — Where durable value sits across the AI stack; Current AI is an attempt to map part of that stack
- [[Local Models in Mid-2026]] — The engineering advances making open models competitive; the Gap Map catalogs the tooling around them
- [[Designing Agentic Loops]] — Simon Willison on the meta-skill of designing toolset + guardrails + success criteria; Datasette Lite as a tool in that loop

---
*Sources: [[summary/open-source-ai-gap-map]]*
*Last updated: 2026-07-05*
