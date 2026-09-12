# Why "Build vs Buy" is the Wrong Question

Chris James argues the familiar build-vs-buy debate is a binary trap that misses the most expensive category of software decisions. Using Eric Evans's Domain-Driven Design subdomain taxonomy (generic, core, supporting), he reframes the question as "what kind of subdomain is this?" — and shows that the third category, supporting subdomains, is where most organizational waste hides.

---

> "You would be foolish to invest the time, risk and so on of building these systems yourself."

On **generic subdomains** (payroll, email, etc.): buy. The expertise exists, the opportunity cost of building is enormous, and you gain nothing competitively. This is the easy call.

> "Build-vs-buy comparisons downplay the cost of integration, because the cost feels invisible down the line."

This is the essay's sharpest observation. The buy decision looks clean on a spreadsheet but materializes as friction — "death by a thousand cuts" — through mapping complexity, vendor upgrade cycles, misleading documentation, and the anticorruption layers every competent team builds around a vendor. None of it appears as a line item. It appears as "why can't engineering ship faster?"

> "Nobody builds their business on solving your exact problem."

**Supporting subdomains** are the third category Evans identified and most build-vs-buy debates ignore. They're not generic enough to buy off the shelf, not valuable enough to be core. When you vendor a supporting domain, you inevitably buy something far larger than your need, paying indefinitely for features you'll never use — then retroactively justifying them. James calls this "a sunk cost you're rationalising before you've even signed the contract."

> "When you own the supporting subdomain, there is nothing to protect your core from. You are the vendor."

The anticorruption layer vanishes. Building it yourself is "simple and finite" — beyond care and maintenance, you're done. The deep irony: trying to avoid investing in a supporting subdomain leads to investing *more*, just in things with zero value to your core business.

> "AI does not replace expertise, it augments it."

On where AI fits: generic subdomains still want purchased expertise, not vibe-coded replacements. Core subdomains need deep domain understanding AI can't substitute. Supporting subdomains — simple, stable requirements — are where AI shines, which James notes is ironic: the same people saying "buy don't build" for supporting subdomains are often the loudest AI boosters, missing that AI most reduces the cost of *building* those exact systems.

---

## Key Themes

- **#concept** — Evans's three subdomain types (generic, core, supporting) as a build-vs-buy decision framework; the third type is where most mistakes happen
- **#pattern** — Integration cost as invisible friction: anticorruption layers, vendor upgrade debt, mapping complexity — real costs that never appear on a purchase order
- **#concept** — Vendor gravitational pull: buying something oversized for your need and paying for the excess indefinitely, then rationalizing it
- **#concept** — "You are the vendor": owning a supporting subdomain eliminates the anticorruption layer entirely; the build is simple and finite
- **#concept** — AI as build-cost reducer for supporting subdomains specifically; doesn't replace expertise in generic domains or deep understanding in core domains

## Critical Analysis

James has written the missing chapter between Evans (2003) and the 2026 AI-everything discourse, and it's better than it needed to be. The three-subdomain frame is genuinely useful — not just as taxonomy but as a decision tool — and he resists the temptation to overclaim. He doesn't say "always build supporting subdomains." He says the build-vs-buy binary misses the category where the question is hardest and the spreadsheet math is most misleading.

The essay's weakness is its silence on *when* to buy a supporting subdomain anyway. Sometimes the build really isn't worth it — regulatory compliance, tiny user bases, extreme time pressure. James acknowledges "the answer won't always be obvious" but doesn't give criteria for the buy case. The JFDI section reads as passionate but underspecified: "simple and finite" describes cleanly scoped supporting subdomains, not the sprawl-prone ones that got vendored in the first place because nobody could agree on their boundaries.

The anticorruption layer argument is the most transferable insight. Every team that has ever wrapped a vendor API in their own interface knows this cost intimately — James just names it and prices it. In the AI era, this argument cuts both ways: if AI makes building supporting subdomains dramatically cheaper, the integration-cost side of the ledger tilts even further toward build. The people James is criticizing — "buy don't build" + "AI goes fast" — are holding two positions that contradict each other, and he catches them at it.

The piece pairs well with [[The Minimum Viable Unit of Saleable Software]] (Brandur's buy-vs-build economics in the LLM era) and [[The Case Against Building Your Own Agent Platform]] (Pete Johnson's build-vs-buy triage for agent infrastructure). Together they form an emerging pattern: the build-vs-buy question keeps getting asked because the wrong framing keeps getting used. Evans gave us the taxonomy 23 years ago. The AI cost shift makes it urgent to actually use it.

A classic case study of this framework in action is the Object/Relational Mapping decision. [[The Vietnam of Computer Science]] is essentially a build-vs-buy autopsy of ORM as a "buy" decision for data access: the integration cost (mapping complexity, the anticorruption layer every team builds around their ORM, the dual-schema problem) is exactly the hidden friction James warns about. Neward's six possible responses map cleanly onto Evans's subdomain types — and his conclusion that no single answer works for every project is James's argument restated: the question isn't "should we use an ORM," it's "what kind of data access does this subdomain actually need?"

---

*Sources: [[summary/why-build-vs-buy-is-the-wrong-question]]*
*Last updated: 2026-07-05*
