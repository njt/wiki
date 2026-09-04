# Domain Storytelling

A collaborative modeling method that uses pictographic sentence diagrams to build shared understanding between domain experts and software teams. Co-created by Stefan Hofer and Henning Schwentner at WPS in Hamburg, it treats workshops as structured conversations where a moderator visualizes domain experts' stories as sentences — who does what with what, and with whom. #tool #pattern #concept

---

## What It Is

Domain Storytelling is a workshop technique for exploring business processes. A moderator facilitates a conversation between **storytellers** (domain experts) and **people with questions** (developers, analysts), drawing a pictographic diagram live as the story unfolds. Each story covers one **scenario** — a concrete example of a business process. The visual language is deceptively simple: actors, activities, and work objects connected into sentences, numbered sequentially.

The method emerged from the University of Hamburg and WPS (a university spin-off) and pre-dates the DDD renaissance. It only got a catchy name around 2015 when Hofer and Schwentner simplified the academic precursor, began presenting at meetups, and broke through in 2018 with talks at every DDD conference plus the release of Egon.io, their open-source modeling tool.

## Key Quotes

> "They [the diagrams] were used as the backbone of the requirements and the domain model."

The diagram isn't decoration — it's the durable artifact that requirements and the domain model hang off. This is the core claim: visualization as epistemology, not documentation.

> "Domain stories are like a puzzle where you have to put the pieces together — in the right order and arrangement."

Henning Schwentner on why layout matters. Domain stories don't use a left-to-right timeline (unlike Event Storming) because they need to show cooperation between actors. Arrows carry the meaning.

> "We modeled three scenarios that covered the existing solution in the first meeting. It uncovered the shortcomings of the existing software and motivated many of the requirements."

Stefan Hofer's war story: three scenarios did what 300 pages of use cases couldn't. A month later the same three scenarios explored to-be processes. Three months after that, they caught a workflow that wasn't documented in the requirements at all. The pattern is potent: a small number of well-chosen scenarios, reused across the project lifecycle.

> "First model a 'pure' version of the business processes… leave out existing systems and focus on what is needed to make the business process work."

Henning Schwentner on finding subdomains. Model what the business actually needs before contaminating the picture with what the current systems happen to do. Activities that belong together because they serve a common goal become subdomains — the starting point for Bounded Contexts.

## Domain Storytelling vs. Event Storming

Stefan Hofer uses both techniques almost equally and argues the choice is situational:

| Dimension | Domain Storytelling | Event Storming |
|---|---|---|
| **Best for** | Many actors, cooperation patterns | Event-driven flows, temporal ordering |
| **Temporal layout** | Free-form, arrow-based | Left-to-right timeline |
| **To-be design** | Easier (sentence-by-sentence moderation converges on shared perspective) | Possible but harder to converge |
| **Iteration granularity** | Micro: one sentence at a time | Macro: larger storming/consolidation cycles |
| **Unit of work** | One scenario | One process |

The two methods aren't competitors — Hofer and Schwentner recommend combining them: Domain Storytelling for broad strokes (purpose, users, scope, domain language), then Event Storming for design-level detail on individual slices.

## The Event Sourcing Gap

Domain Storytelling has no native Event Sourcing support — unlike Event Storming (which has events as a first-class concept) or Event Modeling (which bakes in the full CQRS/ES pattern). Events, commands, and views *can* be modeled as work objects, but the article acknowledges this as a gap. The recommended bridge: use Domain Storytelling for the big picture, then switch to Event Storming for technical design, slicing stories into 1–3 sentence chunks for implementation.

This is honest about a real limitation. Domain Storytelling is strongest where software design methods are weakest — understanding *why* before *how* — and weakest where they're strongest — technical design. It's a complement, not a replacement.

## Workshop Discipline

Henning Schwentner's three rules for a first workshop are deceptively simple and widely violated:

1. **Invite real domain experts, not proxies.** The person who sends someone else "because they're busy" is guaranteeing rework.
2. **Agree on a scenario and return to it.** Workshops drift. The scenario is the anchor.
3. **Agree on as-is vs. to-be and granularity level.** "Let's just explore" is how you get a diagram that's neither accurate nor useful.

## Critical Analysis

**What's genuinely valuable:** The sentence-level granularity — iterating one sentence at a time between storming and consolidation — is the method's killer feature. It prevents the workshop from becoming a free-for-all where the loudest person's mental model wins. The moderator-as-visualizer role forces precision: if you can't draw it, you don't understand it yet.

**What's undersold:** The real-world example buried in the interview is the strongest evidence for the method. Three scenarios caught what 300 pages of use cases missed, and the same scenarios remained useful months later for to-be design and incremental rollout planning. That's not a workshop technique — that's a durable project artifact. The interview treats it as an anecdote when it should be the headline.

**What's missing:** The Event Sourcing gap is honest but the proposed fix (switch to Event Storming for technical design) sidesteps the deeper question: can a scenario-based, cooperation-focused modeling method ever produce good event-driven architectures, or does it naturally bias toward synchronous, request-response thinking? The sentence grammar (subject-verb-object) maps cleanly to commands, but events — things that *happened* — don't have a natural actor in the same way. This isn't a flaw in Domain Storytelling so much as a recognition that different modeling grammars reveal different aspects of a system.

**The AI angle is tantalizing but thin:** The interview mentions that domain stories are being used to feed domain knowledge into LLMs to generate mock-ups and APIs. This is a throwaway line but it points at something significant: a structured, sentence-level domain model is exactly the kind of artifact an LLM could consume to generate implementation scaffolding. If Domain Storytelling's grammar becomes machine-readable (Egon.io already has an open format), it could become a bridge between domain experts and AI-assisted implementation. [[DDD Matters More When AI Writes Your Code]] builds that bridge from the DDD side: Smółka argues the model's value is the team's shared understanding, not an artifact an agent can generate — so a machine-readable domain story is exactly what an agent should consume, and exactly what the team must still own.

**Henning vs. Stefan:** The interview doesn't distinguish their contributions enough. Hofer comes across as the practitioner with war stories; Schwentner as the methodologist who names the patterns. Both are necessary, but the practitioner voice is more persuasive. "We tried it and it worked" beats "here's the theory" every time.

## Related Pages

- [[barnstormer]] — Spec-authoring tool that uses event sourcing with the actor model; Domain Storytelling's sentence grammar maps naturally to barnstormer's event streams
- [[Specifications as the Product]] — The test suite as durable artifact; Domain Storytelling makes the domain story the durable artifact
- [[Software Engineering Craft]] — Hub page for fundamentals that don't change; Domain Storytelling is a requirements craft technique
- [[Engineering for Bounded Cognition]] — Working memory as the constraint; Domain Storytelling's sentence-at-a-time iteration is designed for the ~4-chunk working memory limit
- [[The Joy and Power of Understanding]] — Understanding as pragmatic path and intrinsic reward; Domain Storytelling is a method for achieving shared understanding
- [[Discovery Debt]] — The accumulated weight of untested assumptions; Domain Storytelling is a discovery-debt prevention technique
- [[DDD Matters More When AI Writes Your Code]] — Smółka's argument that DDD's ideas (knowledge crunching, ubiquitous language, design before code) matter more when AI writes the code; domain stories are the shared understanding an agent can't substitute for

---
*Sources: [[summary/domain-storytelling-interview]]*
*Last updated: 2026-07-05*
