# OpenViktor

Open-source "AI employee" platform built in 48 hours by HUMALIKE.AI, launched to #3 on Product Hunt with 300+ GitHub stars in 24 hours, then killed and rebuilt from scratch as the proprietary Jared (jared.so). The blog post detailing this journey is password-protected on Mateusz Jacniacki's site — the narrative is literally gated.

---

## Key Context (reconstructed from secondary sources)

OpenViktor was an "AI employee" you could "hire in 60 seconds" — connect it to Slack, Gmail, Notion, GitHub, Linear, Stripe, give it an email address and Slack identity, and it would read your org's entire history before starting work. Self-driven execution: no prompts needed. Free forever, fully open source.

Then it vanished. The GitHub repo, website, and docs were taken down. The team rebuilt everything as **Jared** (jared.so), pitched as "the first AI employee that's actually social" — proactive monitoring of conversations, 10,000+ integrations. Backed by ElevenLabs' first investor. The pivot from open-source platform to proprietary SaaS is the story, but because the source blog post is password-protected, we can't read Jacniacki's version of it.

- **Team:** HUMALIKE.AI (Spain × Poland). Martí co-founder, Mateusz Jacniacki CTO.
- **Status:** Discontinued. Jared is the successor.

## Key Themes

#tool #startup #pivot #open-source

### The 48-Hour Launch as Market Research

Shipping a fully functional AI employee platform in a weekend and launching on Product Hunt is an extreme version of [[Radical Accountability]]: build something real, get signal from actual users, decide based on data. The 48-hour constraint forced scope decisions that a funded startup would spend months debating. The Product Hunt response (#3 of the day) validated demand; the decision to kill it validated that the approach was wrong.

### Open Source as a Staging Ground

OpenViktor was open source; Jared is not. The pattern is familiar — build in public, learn what works, rebuild with conviction behind a business model. What's unusual is how fast the cycle ran: 48 hours to build, probably weeks to decide to pivot. This is [[Simplicity in the Age of AI-Assisted]] at product scale: build cheap, discard cheap.

### The AI Employee Category

OpenViktor/Jared sits in the same conceptual space as [[Minions — Stripe's One-Shot Coding Agents]] and [[Chief of Staff]]: AI that doesn't wait for instructions. The difference is scope — OpenViktor targeted the whole org (Slack, email, docs, payments), not just engineering. This is the [[Long Live Systems of Record]] thesis in action: the AI employee doesn't replace tools, it sits on top of them.

## Critical Analysis

The password-protected blog post is the most telling artifact here. Jacniacki wrote something about OpenViktor that he's not ready (or not willing) to make public. Given the pivot to Jared, the post is likely a postmortem — what worked, what didn't, why the rebuild. The fact that it's gated while Jared is live suggests one of: (a) the post contains strategic detail the team doesn't want competitors to see, (b) the post is critical of decisions made during the pivot and isn't cleared by the company, or (c) the post was written for a specific audience (investors, early users) and was never meant to be public.

The 48-hour build time is impressive but also a yellow flag. An AI employee that handles email, Slack, payments, and code in 48 hours is almost certainly a thin wrapper around foundation model APIs with minimal safety engineering. The rebuild as Jared suggests the team came to the same conclusion. This is the [[AI Coding Tools Create More Bugs Than They Fix]] problem at the product level: "it works" for demo purposes doesn't mean it's safe for production use in an organization's communications and financial infrastructure.

The "hire in 60 seconds" framing is clever marketing — it positions the AI as an employee rather than a tool, which is the correct mental model per [[Building Agents for Production Systems with MCP]]. But "hiring" an AI that you can't fire, can't hold accountable, and that has access to your entire org's data history is a trust problem the product almost certainly hadn't solved. The pivot to Jared's "social AI" framing suggests the team learned that adoption requires trust-building, not just capability.

The open-source-to-proprietary pivot is worth noting but not damning. The team used open source as a launch strategy, got signal, and moved on. That's a legitimate approach. What's more interesting is whether any of the OpenViktor code lives on in forks or derivatives — the 300+ GitHub stars suggest interest, but a repo that's been deleted can't be forked. This is the risk of building on open-source tools that can disappear: you're not just betting on the code, you're betting on the maintainer's continued interest.

## Cross-Links

- [[Minions — Stripe's One-Shot Coding Agents]] — same "AI that just does the work" category
- [[Chief of Staff]] — proactive AI that doesn't wait for prompts
- [[Long Live Systems of Record]] — AI employees sit on top of tools, don't replace them
- [[Radical Accountability]] — 48-hour build as extreme ownership
- [[Simplicity in the Age of AI-Assisted]] — build cheap, discard cheap
- [[AI Killing B2B SaaS]] — AI employees as the SaaS replacement thesis
- [[Building Agents for Production Systems with MCP]] — the integration pattern OpenViktor was built on
- [[AI Coding Tools Create More Bugs Than They Fix]] — the "works for demo" vs "safe for prod" gap
- [[Smart Models Dumb Pipes]] — the architectural question: where does the intelligence live?
- [[Two Kinds of User Are Emerging]] — Product Hunt launch dynamics and user segmentation

---
*Sources: [[raw/openviktor]] (reconstructed from secondary sources — blog post password-protected)*
*Last updated: 2026-05-15*
