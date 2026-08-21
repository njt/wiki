# Software Engineering Practice Atlas

A comprehensive, AI-generated reference map of software engineering craft: 4,654 entries spanning five practice areas (Engineering, Engineering Management, Product Management, Project/Program Management, Research) and 25 domain-specific field guides. Each card follows a disciplined format — what it is, why it matters, how to apply it, when *not* to use it — and links to related cards, sources, and crawled engineering blog articles. The site is unsigned, the content is AI-generated and not yet copy-edited, and the project is honest about both its ambitions and its limitations.

---

## Key Quotes

> "An attempt to gather the craft of building software in one place. The practices, principles, patterns, and metrics that experienced engineers, product managers, and leaders reach for are scattered across countless books, talks, and blog posts. This collects them into a single map you can wander through, rather than stumble upon one article at a time."

The pitch is strong and the map metaphor is deliberate: this isn't a textbook to be read linearly, it's a navigable space. The verb "wander" is doing real work here — it promises serendipity, not just search. This is the same promise that made Wikipedia a revolution and makes most reference sites feel like filing cabinets.

> "Every entry says what it is, why it matters, how to apply it, and — the part most references skip — when not to use it."

The "when not to use it" field is the differentiator. Most engineering references tell you what a practice is good for; almost none tell you when it's actively harmful. This is the difference between a catalog of tools and a catalog of *judgment*. It's also the hardest field to generate well with AI, which makes it the most interesting test of the site's quality.

> "Most content is AI generated and has not yet been copy-edited by a human, and some domains are less balanced or complete than others. Treat it as a map to explore from, not gospel, and verify before you rely on it."

The honesty is disarming and strategically smart. By preempting the obvious criticism, the site frames AI generation as a starting point rather than a finished product. But the admission also raises the question the site can't answer: if you need to verify everything, what value does the synthesis add over searching the original sources directly?

## Key Themes

#tool #reference #pattern-catalog #software-craft #ai-generated #map

### The Pattern Language Lineage

The atlas sits in a direct line from [[A Pattern Language (Christopher Alexander)]] — the 1977 book that gave architects 253 composable patterns for buildings and towns. Alexander's insight was that good design isn't invented from scratch; it's assembled from known solutions to recurring problems. The Practice Atlas applies the same model to software engineering, but at a scale Alexander couldn't have imagined: 4,654 cards versus 253 patterns.

The card format itself is an evolution. Alexander's patterns had a fixed structure (name, context, problem, solution, diagram, related patterns). The Practice Atlas keeps the structure but adds the "when not to use it" field — a genuinely novel contribution that Alexander's patterns lacked. This alone makes it worth browsing, even with the AI-generation caveat.

It also invites comparison with [[Agentic Design (Pattern Catalog)]], KORTEXYA's 280+ pattern catalog for AI agent architecture. Both are pattern-language revivals in the software domain. The difference is scope: KORTEXYA goes deep on one domain (agent architecture), the Practice Atlas goes broad across the entire software craft. The KORTEXYA catalog is presumably human-curated; the Practice Atlas is explicitly AI-generated. Which approach produces better patterns is an open empirical question, and the two projects together form a natural experiment.

### The Five-Map Structure

The five maps — Engineering, Engineering Management, Product Management, Project/Program Management, Research — are a taxonomy worth examining:

**Engineering** (1,512 cards) is the largest applied map, which makes sense: it covers the full lifecycle from design through operations. Its deep content is a ten-chapter guide walking those ten quality attributes; the opening framing and first chapter are unpacked in [[The Ten Properties of Software Quality]]. **Research** (2,113 cards) is the largest overall, and its presence as a co-equal map is the most interesting editorial decision. Most engineering references treat research as a separate discipline or ignore it entirely. Including it alongside engineering management and product management makes a claim: that research methodology is core craft knowledge for software engineers, not an academic specialty. This aligns with [[Five Studies That Are Changing How I Think About AI in Software Engineering]] and the broader argument that empirical reasoning is becoming a core engineering skill.

The **Engineering Management** (331 cards) and **Product Management** (414 cards) maps are notably smaller. This might reflect the actual distribution of written knowledge (there are more books about engineering than about managing engineers) or it might reflect the limits of AI generation (management patterns are harder to extract from blog posts than technical patterns). The asymmetry is worth noting: if you're an EM or PM, the atlas is thinner for you.

**Project & Program Management** (284 cards) is the smallest map. This is the discipline that [[When the Target Keeps Moving]] and [[Hidden Inefficiencies Behind Delivery Delays]] are trying to fill. The thinness might be accurate — project management as a written discipline is underdeveloped relative to its importance — or it might be a gap the AI couldn't fill from available source material.

### The Domain Guides Strategy

The 25 domain guides are the site's most architecturally interesting feature. Rather than building separate maps for each domain, they walk the same quality attributes as the Engineering map but only where the domain genuinely reshapes them. This is smart information architecture: it avoids duplication while acknowledging that domain context changes engineering choices. A reliability pattern that works for web services might not apply to mobile; a testing strategy for distributed systems might be overkill for data pipelines.

The domain selection reveals editorial priorities. The applied domain guides (Distributed Systems, Data, ML/AI, Web, Mobile, Backend/Services, Enterprise, Cloud Infra) are short — 12–20 minutes each. The research domain guides (Algorithms, Formal Methods, Security, Databases, Robotics, etc.) are longer — 26–38 minutes. The site is more comfortable going deep on research domains than applied ones, which again reveals the author's bias toward computer science over software engineering.

### The AI Generation Question

The site is transparent about being AI-generated. This is both a strength and a weakness:

**The case for AI generation**: At 4,654 entries, no human team could produce this breadth at this speed. The AI can synthesize across thousands of sources and maintain consistent formatting across entries. The "when not to use it" field is precisely the kind of structured judgment that LLMs can approximate well enough to be useful — you get 80% of the value of expert curation at 1% of the cost.

**The case against**: The site admits the content hasn't been copy-edited by a human. At this scale, there will be plausible-but-wrong entries — the most dangerous kind. An incorrect "when not to use it" recommendation could steer an engineer away from a practice that would actually help them. The depth varies between entries, and some domains are "less balanced or complete than others." Without knowing which domains those are, the reader is navigating with an unreliable compass.

The practical advice — "treat it as a map to explore from, not gospel" — is correct but insufficient. A map with unknown errors is worse than no map, because it gives false confidence. The site would benefit enormously from some signal of entry quality: a confidence score, a "human-reviewed" badge, or even just an indication of how many sources each card draws from. As it stands, the reader has to bring their own judgment to every card, which limits the atlas's value as a reference.

This connects to the broader tension in [[The New Software Lifecycle]]: AI compresses some parts of the SDLC dramatically while leaving others stubbornly human. The Practice Atlas is an example of AI doing the compression (synthesizing 4,654 entries from scattered sources) while leaving the verification to humans. Whether that's a good trade depends on whether the synthesis saves more time than the verification costs.

### Where It Fits

The Practice Atlas occupies an interesting niche. It's not a textbook (no narrative arc, no exercises). It's not a wiki (no collaborative editing, no discussion). It's not a search engine (no indexing of the open web). It's a curated map — and "curated by AI" is a genuinely new category, however uncomfortable.

The same site also exposes its raw material directly: the [[Engineering Atlas Articles]] index — 5,172 crawled engineering reports classified along three axes (outcome, phase, quality attribute), where the cards' linked "crawled engineering blog articles" actually live. It is the evidence layer beneath this synthesis layer, and its distribution is telling: lessons (2,734) outnumber wins (1,620) and outages-plus-postmortems (688) combined, while correctness (2,981) is the most-tagged attribute.

In the wiki's landscape:
- [[Software Engineering Craft]] is the hub for fundamentals, and the Practice Atlas is essentially an external reference that covers much of the same territory in card format
- [[Patterns.dev]] covers web design and rendering patterns specifically; the Practice Atlas covers those plus everything else
- [[The New Software Lifecycle]] maps how AI compresses the SDLC; the Practice Atlas is a product of that compression applied to reference material
- [[A Pattern Language (Christopher Alexander)]] is the intellectual ancestor; the Practice Atlas is the most ambitious software-domain descendant
- [[Command Line Interface Guidelines]] is an example of the kind of concentrated craft knowledge the Practice Atlas distributes across thousands of cards

The site's greatest potential value is as a map for AI coding agents themselves. If each card is a structured judgment about when to use and not use a practice, that's precisely the kind of context that could improve agent decision-making. The atlas could become a context source that agents consult before making architectural choices — a reference implementation of the harness-engineering thesis from [[The New Software Lifecycle]] and [[Loop Engineering]].

### The Unsigned Problem

The site has no author, no About page, no indication of who built it or why. This matters for two reasons. First, trust: in a field where the best resources carry personal signatures ([[Dan Luu on AI Coding]], [[The New Software Lifecycle]], [[Elements of Agentic Systems Design]]), anonymity is a trust deficit. Second, sustainability: without knowing who maintains it, there's no way to assess whether the content will be updated, corrected, or abandoned. The site could be a weekend project that's already finished, or the beginning of a sustained effort. There's no way to tell.

This is the same problem I noted with [[Agentic Design (Pattern Catalog)]] and KORTEXYA. The pattern-catalog form seems to attract anonymous or institutionally-opaque producers. Maybe that's because producing a high-quality pattern catalog at scale is inherently a team effort, and individual attribution gets messy. Or maybe it's because the pattern catalog is an appealing form for AI-generated content — structured enough for LLMs to produce consistently, broad enough to mask shallowness, and valuable enough to attract traffic — and the producers don't want their names attached to something they can't fully vouch for.

---

*Sources: [[raw/eng-atlas]]*
*Last updated: 2026-08-01*
