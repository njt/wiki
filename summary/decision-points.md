---
url: https://narphorium.com/blog/decision-points/
title: "Optimizing for Decision Points"
author: Shawn Simister
date_fetched: 2026-07-05
date_published: 2026-03-11
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

# Optimizing for Decision Points

**Author:** Shawn Simister
**Published:** March 11, 2026
**Reading time:** 14 minutes

## Opening

Simister describes how his workflow evolved from chat-based AI assistance toward long-running agents that independently prototype entire ideas. As agents grew more capable of completing larger projects autonomously, he found himself writing more ambitious specs. However, this created a challenge: reviewing more code to determine whether the agent had faithfully executed his intent. Simple features could be verified visually, but harder ones created a strange loop where verifying the agent's output required already knowing the code well enough to inspect it properly.

This led him to reconsider his approach. Having the agent build from a spec and then verifying the result as a separate step made things unnecessarily difficult. He realized that if verification was inevitable, it should be integrated into the spec from the start. This pushed him to think more carefully about which decisions could be delegated to AI and which required his direct involvement.

He notes that while current models can fix bugs in a loop until code executes, that doesn't guarantee the agent built what was asked. The real focus should shift from iterating on code to iterating on ideas. Anyone using the same models can go from a half-formed idea to a working prototype quickly. What differentiates work is the willingness to push beyond what models produce by default. Conventional design choices can be delegated, but the choices reflecting taste and innovation cannot. Models revert to the safest, most conventional approach at every step, and these choices compound — so outcome quality depends on engaging with the moments that truly shape results rather than letting the model decide by default.

## What Are Decision Points?

Decision points are described as the critical moments in a project where human input has outsized impact on direction. "A few minutes of human judgment at the right time saves hours of agent work heading in the wrong direction." Simister acknowledges that developers may recognize this concept from gates and approval processes in traditional SDLC, but notes those often imply sequential sign-offs that block progress. The future involves running many agents in parallel across multiple avenues of exploration, requiring a less rigid approvals model — not just minimizing risk but also maximizing exploration. He references his earlier post "Top-Down vs. Bottom-Up Development" on this tradeoff.

Different types of work need different levels of human involvement, and finding where judgment is needed requires deliberate workflow analysis. Some work is obvious enough to describe and delegate fully. But when real ambiguity exists about direction, sending an agent straight to implementation means it fills uncertain decisions with its own defaults — which converge on safe, generic options rather than innovative ones. Brainstorming and planning address this at different levels: brainstorming explores the idea space to sharpen intent before figuring out how to build; planning explores the design space by clarifying scope and architecture before code is written. Task decomposition helps steer implementation of complex plans. Each level provides an opportunity to confirm assumptions before proceeding.

Choosing the right amount of specification and planning is itself a judgment call. Getting it wrong doesn't cost much time since the agent handles planning work, but it costs attention. Over-specifying trivial tasks creates decision points that demand attention without benefiting from taste or judgment. Skipping planning for complex work means losing the chance to verify assumptions before they become baked into code.

A key quote from Ryan Lopopolo (OpenAI) on technical debt and taste is referenced: the idea that human taste, once captured, should be enforced continuously on every line of code, catching bad patterns daily rather than letting them spread.

Even with the right balance, agents uncover unanticipated ambiguities. Workflows must allow new decision points to surface naturally. Simister shares an example of building a mind map app where he chose the dagre library for graph layout. During implementation, the agent hit CommonJS import issues in an ESM Next.js project. The agent defaulted to adding shims and workarounds — solving at the implementation level. But the real decision point was at the package choice level: the agent had defaulted to the old, unmaintained dagre package, while the newer @dagrejs/dagre supports ESM natively. Asking "are we even using the right package?" resolved the issue at its origin.

## Teaching Agents What to Escalate

Claude Code's `AskUserQuestion` tool exemplifies how decision points work in practice. The agent encounters ambiguity, sends a specific question to the human, and continues once it receives an answer. This pattern works because recognizing a decision is needed is simpler than making the decision. The agent doesn't need to know which path is best to notice that multiple valid paths exist and the choice depends on context it lacks. Matching tasks against a taxonomy of common decision points is something agents can do reliably, even when the underlying decisions require expert judgment.

Not every question needs human input. Decisions with clear answers based on established patterns can be resolved by the agent autonomously, so the questions that do reach the human carry more weight. Once a plan is broken into tasks, new decision points emerge — each task can be reviewed before implementation, allowing priority reordering and catching assumptions that didn't hold up when details were filled in.

The more one pays attention to recurring decision points, the more patterns emerge: preferring ESM-compatible libraries, prototyping before writing tests, using streaming events instead of shared state. These become a reusable library of design patterns the agent can apply autonomously, freeing attention for genuinely new decisions where taste matters precisely because no pattern exists yet. "The agent's job isn't to have taste on your behalf, but to notice the moments where taste matters and put them in front of you."

## Designing Workflows Around Judgment

Running multiple Claude Code tasks in parallel, Simister found the hard part wasn't coding itself but knowing where his attention should go. Ideas could be explored in parallel, but everything was spread across terminal sessions and markdown files he had to track mentally.

This led him to build [atelier.dev](https://atelier.dev), an AI-native kanban board grown from his personal Claude Code workflow. Instead of scattered files and windows, tasks move through stages that surface moments where his input matters, providing a single view of what's running, what's blocked on a decision, and what's ready to verify. Each transition between columns calls a custom agent skill to ensure it operates at the correct level of abstraction.

Building Atelier shifted his thinking from verifying AI-written code to considering the cascade of decision points that produce it. Even when an agent appears to fix real problems, those problems exist within a frame created by earlier decisions the same agent made. Reframing a problem at a higher level is often needed to prevent design debt from compounding.

Simister references Donella Meadows' article "Leverage Points: Places to Intervene in a System," which ranks interventions by their actual ability to change system behavior. At the bottom are parameters — tweaking numbers within an existing structure. At the top are system rules, information flow structures, and the mental models driving everything. Her key finding is that people overwhelmingly focus on parameter-level changes, which rarely alter how a system actually behaves.

Code review, for example, catches real problems — style inconsistencies, error handling gaps, design drift — but operates mostly at the parameter level because the frame was already set by earlier design decisions. "The review feels productive, and it is, but the decisions that had the most influence over what you're reading happened before any code was written." Code quality still matters in agentic development, but in the way Meadows' lower-leverage interventions matter: genuinely useful but unable to alter the system's overall direction.
