# ADR (GitHub adr Organization)

The home page of the `adr` GitHub organization — the canonical vocabulary page for Architectural Decision Records: what counts as an Architectural Decision, what an Architecturally Significant Requirement is, and pointers to the practice's history from Michael Nygard's 2011 blog post onward.

---

This is a hub page, not an essay, but it is where the ADR practice defines itself. Three definitions carry everything: an **AD** is a justified design choice addressing an architecturally significant requirement; an **ADR** captures that decision *and its rationale*; the project's accumulated ADRs form its **decision log**. The framing is explicitly part of Architectural Knowledge Management, but the authors deliberately generalize — ADR usage "can be extended to design and other decisions ('any decision record')". The three aims are modest: common vocabulary, better tooling supporting agile and incremental processes, and pointers to public knowledge.

The interesting part for this wiki is the lineage. The page anchors itself in Zdun et al.'s *Sustainable Architectural Decisions* (source of the Y-statement format), Nygard's 2011 post that popularized the template, and a WICSA 2015 paper comparing seven templates — then points at mainstream adoption (Azure Well-Architected Framework, AWS Prescriptive Guidance, a 2022 IEEE Software column). ADRs are a pre-agent engineering practice that has become newly load-bearing now that machines need to read the why.

---

> "An Architectural Decision Record (ADR) captures a single AD and its rationale; Put it simply, ADR can help you understand the reasons for a chosen architectural decision, along with its trade-offs and consequences."

The emphasis on *rationale* and *consequences* is what separates an ADR from a changelog or a commit message. It is the difference between knowing what changed and knowing what was weighed against what.

> "The collection of ADRs created and maintained in a project constitute its decision log."

The decision log as a first-class project artifact — not documentation but a record — is the concept that agent-era tooling has picked up and run with.

> "ADR usage can be extended to design and other decisions ('any decision record')."

A quietly important generalization: the format outgrew architecture. Anything where the why is perishable can be recorded this way.

## Key themes

- **Rationale over artifact** — an ADR's value is the why and the trade-offs, not the decision itself. #concept
- **Decision log as durable artifact** — the accumulated record is the product, matching this wiki's spec-as-artifact framing. #pattern
- **Y-statements and templates** — the practice inherits structure from *Sustainable Architectural Decisions* and Nygard's lightweight template. #tool
- **Mainstream adoption** — Azure and AWS documentation now recommend ADRs; this is no longer fringe practice. #pattern

## Analysis

This page is the establishment view of decision records: definitions, lineage, and adoption signals, with no argumentative edge. Its value here is as the fixed reference point other pages orbit. Read against the agent-era material in this wiki, two things stand out.

First, the pre-agent definition already solves the problem agents create. An ADR is a *justified* choice — it records the rejected alternatives and the consequences. That is exactly the context an agent needs when it revisits code months later, and exactly what a commit message does not carry. The page's "any decision record" generalization anticipates the explosion of agent-written decision artifacts.

Second, the vocabulary is older than the tools. Nygard's post is from 2011; the seven-template comparison from 2015. The practice did not need LLMs to be useful — but it has been repurposed, at speed, by people wiring ADRs into agent loops and skill files. The hub page does not discuss agents at all, which is precisely what makes it useful as a baseline: whatever an agent-era ADR tool adds or drops, you can check against the original definition.

## Related pages

- [[Coding Agents Love Decision Records]] strengthens this page's case from the agent side — argues that agents are the reason ADRs finally took off, since a machine that reads the why can honor it.
- [[ADRs4AI (Deliberative Programming)]] builds directly on this vocabulary, staking decisions as question-format records that keep the why resumable across agent sessions.
- [[Bag of Decisions (Doug Turnbull)]] complicates the tidy "decision log" ideal — a pile of individually recorded decisions still needs structure to be navigable.
- [[Capturing Why Engineering Decisions]] covers the practical debate this page's reading list gestures at: lightweight records vs. heavyweight justification.

---
*Sources: [[raw/adr-github-io]], [[summary/adr-github-io]]*
*Last updated: 2026-10-26*
