---
url: https://www.autoencoders.com/papers/reinvent-software-tickets-for-the-ai-era/
date_fetched: 2026-11-17
---

# It's Time to Reinvent Software Tickets for the AI Era

How we use Linear tickets to preserve decisions, orchestrate coding agents, and ship one parent pull request.

Most ticketing systems were built around a simple question: *Who’s working on what?* When coding agents do much of the implementation, that’s no longer the most interesting question.

At AskFutures, we use Linear extensively. Our tickets have three jobs: **remember decisions, orchestrate parallel agents, and define what ships together.**

## Record decisions, not activity

Fast development produces a constant stream of decisions: how a feature behaves, what gets shown in the UI, which architectural tradeoffs are acceptable. If those decisions live only in an agent’s context, they disappear when the session ends.

At AskFutures, we record the important ones in Linear tickets. Future agents can recover not just *what* was built, but *why*. We don’t create tickets for every internal step an agent takes. We create them for decisions and meaningful deliverables.


Code remembers what changed. Tickets remember why.

## Turn the ticket tree into an execution graph

We create a parent ticket for a feature, divide it into child tickets, encode dependencies, and assign children to coding agents. Independent children run in parallel; blocked children wait.

This changes how we split work. A good child ticket isn’t necessarily a small task. It’s a piece of work an agent can complete with minimal coordination and minimal conflict with other agents.

## One parent ticket. One PR.

This is where the workflow saves serious time. We have a mode where several child tickets feed into **one pull request for the parent**.

When a merge gate is slow, one PR per child is expensive. Grouping work under a parent reduces repeated full-gate runs without giving up parallel development. Integration still matters; we simply avoid paying the largest validation cost for every small task.

## Finding decisions matters more than storing them

Recording decisions is easy. Finding the right one months later is harder. At AskFutures, we solve this with a small text file that acts as a **decision index**.

The file contains short, searchable descriptions and pointers—not copies of decisions. Agents follow three levels of progressive disclosure: read the small map, open relevant Linear tickets, then inspect code or PRs only if needed.

The descriptions are written for discovery. “Search layout, pagination, chart height, mobile navigation” is easier to find than “pagination.” Linear remains the source of truth; our text file is only the map.

## The system in one paragraph

**Our system at AskFutures is simple:** Linear tickets preserve decisions. Parent-child tickets and dependencies coordinate agents. One parent PR gives us an efficient integration boundary. A lightweight text index makes the history discoverable.

We don’t use tickets to count how busy agents are. **We use them to keep parallel work coherent.**
