---
url: https://mattwynne.net/lean-software-production
title: Lean Software Production
author: Matt Wynne
date_fetched: 2026-07-05
date_published: 2026-05-16
site: mattwynne.net
topics:
  - agent-coding-workflow
---

# Lean Software Production

Matt Wynne proposes a new conceptual framework called **Lean Software Production** to describe how AI is transforming software development. Software is "no longer a craft" but mass production approaches are "too rigid, inflexible, and inhumane." The framework combines three pillars: Lean thinking, Extreme Programming discipline, and agentic production systems.

## The Three Pillars

1. **Lean** — human-centric continuous improvement, systems thinking, and pull-based flow from manufacturing
2. **Software** — XP engineering disciplines that "keep the software malleable"
3. **Production** — agentic orchestration and dark factory patterns where LLMs generate code unattended

## Why Lean?

Wynne draws on W. Edwards Deming's toast-burning/scraping analogy to critique how many currently use LLMs — generating code then inspecting quality through review. He advocates adopting *kaizen* (continuous improvement) and defect prevention instead.

Key practices proposed:
- Instead of blaming the model for mistakes, improve the context it receives
- Capture architectural decision records (ADRs) and design heuristics in the repo for agents to study
- Have agents present information in accessible formats (slide decks, HTML summaries) rather than walls of text

Wynne cites the Toyota Production System as a socio-technical model, referencing "an elaborate dance between humans and machines." The lean concept of *Jidoka* (building quality in) means focusing on "crafting the system we use to produce reliable, well-engineered software at scale."

References: Deming, Toyota Production System, The Goal (Goldratt), Lean Software Development (Poppendieck), ADR documentation, Rebecca Wirfs-Brock's design heuristics.

## Why Extreme Programming?

When "a single engineer can generate the volume of output that once required a team," problems like ambiguity, architectural weakness, and unreliable tests become catastrophic — they "set things on fire." XP practices become "standard industrial safety equipment."

Practices highlighted:
- **Pervasive automated testing** — compared to "double-entry bookkeeping for your code"
- **Continuous integration** — for fast feedback on integration
- **Relentless refactoring** — keeping software "soft and malleable"
- **Pairing on decision-making** — maintaining shared mental models through conversation

Concrete examples: robot-introduced mutation tests in CI, autonomous review/fix loops adopting personas like Sandi Metz or Martin Fowler, agents looping to identify and optimize slow tests.

Wynne distinguishes between using agents for TDD (which he considers "largely been solved") versus truly adopting XP values for the agentic era.

## Towards the Lean Software Factory

Wynne recounts a workshop with Justin McCarthy (co-founder/former CTO of StrongDM), who pioneered the "dark factory" concept. McCarthy's team rule: "Code must not be written by humans" and "Code must not be reviewed by humans." This constraint forces teams to build systems that can judge code quality autonomously.

Wynne criticizes a "half-way house" approach where people use LLMs to generate code without modernizing the entire delivery pipeline.

He introduces **Annie Vella's "middle loop"** concept — a layer where engineers supervise AI doing previously manual work — and notes mixed results due to lack of established patterns. He references **Birgitta Böckeler's "harness engineering"** where engineers focus less on coding and more on engineering the *system that produces the code*.

The core question: "what *can't* the agents do?" Continuously asking this allows teams to automate mundane work and focus on deciding "what problems to solve, and how to judge that they were solved."

Final framing: "The product is still working software, but now the work is engineering the system that produces it."

## Acknowledgments

The post was "entirely organically written by my human hands and brain." Feedback credited to Jeremy Lightsmith, Rob Bowley, Chris Parsons, Emily Bache, and Dave Farley. Published via write.as.
