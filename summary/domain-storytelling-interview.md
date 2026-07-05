---
url: https://docs.eventsourcingdb.io/blog/2026/07/02/use-model-with-domain-an-interview-on-domain-storytelling/
title: "Use Model With Domain: An Interview on Domain Storytelling"
author: Golo Roden (interviewer), Henning Schwentner & Dr. Stefan Hofer (interviewees)
date_fetched: 2026-07-05
date_published: 2026-07-02
site: EventSourcingDB Blog
category: Interview
---

# Use Model With Domain: An Interview on Domain Storytelling

## Interview Participants

- **Golo Roden** — CTO and founder of the native web (EventSourcingDB). Conducts the interview.
- **Henning Schwentner** — Coder, Coach and Consultant at WPS – Workplace Solutions (Hamburg, Germany). Co-creator of Domain Storytelling.
- **Dr. Stefan Hofer** — Software developer and consultant at WPS – Workplace Solutions (Hamburg, Germany). Co-creator of Domain Storytelling.

## Origins

Domain Storytelling's predecessor was developed at the University of Hamburg and WPS, a university spin-off. When Stefan Hofer joined WPS in 2005, colleagues were already using the method in client requirements workshops. The diagrams served as the backbone of requirements and domain models.

Henning Schwentner notes that the method felt unique until DDD gained adoption around 2015. Around that time, they simplified the academic method, gave it a catchy name, and began presenting at meetups. 2018 was their breakout year — talks and workshops at all DDD conferences plus the release of Egon.io, their open-source modeling tool.

## Visual Language

The pictographic language was inspired by Rich Pictures. Henning describes the structure as "sentences with a subject, verb, and objects" — specifically *who* (actor) does *what* (activity) with *what* (work object) with *whom* (another actor). A *story* consists of multiple sentences ordered by sequence numbers.

## Workshop Mechanics

A typical workshop involves:
- **People with questions** (developers, business analysts) and **storytellers** (domain experts)
- A structured conversation led by a **moderator**, who visualizes the conversation live as a diagram
- Storytellers address one **scenario** at a time — a concrete, meaningful business process example
- Only a few domain stories are typically needed to understand a business process

## Comparison with Event Storming

Stefan Hofer uses both techniques almost equally; the key is that either facilitates conversation and shared visualization. Domain Storytelling is preferred when the domain involves **many actors** (people or software) and you want to examine **how they cooperate**. It's considered easier for designing to-be processes because a common perspective emerges from a moderated, sentence-by-sentence approach.

Layout difference: Domain stories don't follow a left-to-right timeline because they need to show cooperation between actors — this works better with arrows. Each domain story covers one scenario. In Domain Storytelling, iterations of storming (discussing the next sentence) and consolidation (recording that sentence) are much smaller than in Event Storming.

## Real-World Example

Stefan Hofer described a project implementing a requirements specification: 80 use cases, 300 pages. The use cases proved insufficient for truly understanding the domain. Stefan modeled three scenarios covering the existing solution in the first meeting, revealing shortcomings that motivated many requirements. A month later, the same three scenarios were used to explore to-be processes, clarifying the interplay of use cases. Three months later, with several use cases implemented, they faced an incremental rollout requiring the old and new systems to collaborate. Domain Storytelling revealed that the newly designed software-supported workflow was not documented in the requirements. New requirements were derived by transcribing activities from the domain story.

## DDD & Event Sourcing Connections

### Bounded Contexts
Henning recommends first modeling a "pure" version of business processes, leaving out existing systems and focusing on what's needed to make the process work. Domain experts are then asked which activities belong together because they serve a common goal. Activities that belong together are visually grouped. The result is a Subdomain — a starting point for Bounded Contexts.

### Event Sourcing
Domain Storytelling does not have "native" Event Sourcing support like Event Storming or Event Modeling. However, events, commands, and views *can* be modeled as work objects. A recommended approach combines Domain Storytelling with Event Storming:
1. Use Domain Storytelling for broad strokes — purpose, users, system interactions, domain language, scope
2. Slice the story for development (typically 1–3 sentences per slice)
3. Do design-level Event Storming for the first slice, implement it, then repeat

## Tools

- **Egon.io** — Open-source browser-based modeling tool. Free, no account required, no tracking, no data storage on their servers.
- **domainstorytelling.org** — Getting-started resource.
- **Book** (Addison-Wesley) — Published at domainstorytelling.org/book.
- Community contributions include PlantUML support and an early VS Code extension based on Egon. Mermaid support is rumored to be next.

## Top Three Tips for a First Workshop (Henning Schwentner)

1. Invite **real domain experts**, not proxies.
2. **Agree on a scenario** and return to it if discussion drifts.
3. **Agree on as-is vs. to-be** modeling and on granularity level (coarse overview, fine details, or in between).

## Future Directions

Egon is under active development. AI is a major driver of ideas — domain stories are being used to feed domain knowledge into LLMs, e.g., to generate mock-ups and APIs.
