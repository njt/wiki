---
title: "Gas Town's Agent Patterns, Design Bottlenecks, and Vibecoding at Scale"
url: https://maggieappleton.com/gastown
author: Maggie Appleton
date_fetched: 2026-05-15
date_published: 2026-02
---

# Gas Town's Agent Patterns, Design Bottlenecks, and Vibecoding at Scale

Maggie Appleton's analysis of Steve Yegge's Gas Town — a Mad-Max-themed agent orchestrator running dozens of coding agents simultaneously. Yegge built it in "17 days, 75k lines of code, 2000 commits." Appleton treats it not as a working tool (it isn't) but as "a good piece of speculative design fiction" that sketches future agent orchestration patterns.

Gas Town is entirely vibecoded, burns thousands monthly in API costs, and Yegge himself warns people not to use it. Yet Appleton argues it should be taken seriously because the patterns buried in the chaos — specialized agent roles, ephemeral sessions with persistent state, continuous work queues, agent-managed merge queues — are early sketches of how multi-agent development systems will work.

**Note: The source article was truncated during fetch; Sections 3 (price/value analysis) and 4 (six factors for when to stop looking at code) were not retrieved.**

## Introduction

Appleton frames Gas Town as design fiction: "creating objects from a plausible near future to provoke questions, not predict." She praises Yegge for "exercising agency and taking a swing" despite the mess. Her key disclosure: she's in "the agentically conservative camp" (stages 4-6 of Yegge's 8-level automation scale), reviews diffs carefully, and hasn't used Gas Town in earnest.

The $GAS meme coin (doing $400k+ on Bags) is noted as "purest of pure speculative betting" — not Yegge's doing but a sign of the hype around autonomous agent systems.

## Section 1: Design and planning becomes the bottleneck when agents write all the code

When agents handle implementation, development time is no longer the limiting factor. Yegge: Gas Town "churns through implementation plans so quickly that you have to do a LOT of design and planning to keep the engine fed."

Appleton confirms this from personal experience: "the build time is rarely what holds me up. It is always the design." Agents can't make decisions about human context, taste, preferences, and vision.

The biggest flaw: Gas Town is "poorly designed." Yegge didn't design the system ahead of time — he "just made stuff up as he went." Yegge's own admission: "Gas Town is complicated. Not because I wanted it to be, but because I had to keep adding components until it was a self-sustaining machine."

HN commenter qcnguy: Beads was "a stream of consciousness converted directly into code" — "not only vibe coded, it was vibe designed too." Gas Town is "the same thing multiplied by ten thousand."

Bluesky's astrra.space: the mayor is "dumb as rocks," the witness "regularly forgets to look at stuff," polecats "seem intent on wreaking as much chaos."

Appleton's warning: "This thing fits the shape of Yegge's brain and no one else's." The footgun: moving so fast you never stop to think, building without considering each step, ending up "hip-deep in poor architectural decisions" having "burned a billion tokens in exchange for a pile of hot trash."

## Section 2: Buried in the chaos are sketches of future agent orchestration patterns

Despite the mess, Appleton finds four useful patterns:

### Specialized roles with hierarchical supervision

Every agent has a permanent, specialized role. The Mayor (human concierge, never writes code), Polecats (temporary grunt workers), The Witness (supervises Polecats), The Refinery (manages merge queue, resolves conflicts). Plus the Deacon, "Boot the Dog," and a crew of "dogs" doing maintenance.

This hierarchy "solves both a coordination and attention problem." With the Mayor as single interface, coordination overhead disappears.

### Persistent roles and tasks, ephemeral sessions

Each agent session is "disposable by design." Identity and tasks are stored in Git. Sessions are killed and fresh ones spun up. "Seancing" lets new agents ask predecessors about unfinished work.

This is implemented through **Beads**: tiny trackable work units stored as JSON in Git alongside code. Each bead has an ID, description, status, and assignee. Agent identities are also stored as beads.

Appleton notes Anthropic described the same approach in their November 2025 research on "effective harnesses for long-running agents."

### Continuous streams of work

Gas Town is "a perpetual motion machine." The Mayor breaks features into atomic tasks. Each worker has its own queue and a "hook" pointing to current work. When one task finishes, the next jumps up. Workers are never idle — as long as you keep feeding the Mayor.

### Merge queues and agent-managed conflicts

The Refinery agent handles merging and can creatively re-implement when conflicts arise while preserving original intent.

[Content truncated — Sections 3 and 4 not retrieved]

## Section 3: The price is extremely high, but so is the (potential) value

[Not retrieved from source]

## Section 4: Yegge never looks at code. When should we stop looking too?

Six factors planned:
- Domain and programming language
- Access to feedback loops and definitions of success
- Risk tolerance for shit going wrong
- Greenfield vs. brownfield projects
- Number of collaborators
- Your experience level

[Content not retrieved from source]
