---
url: https://www.oreilly.com/radar/coding-agents-love-decision-records/
date_fetched: 2026-10-03
---

The following article originally appeared onDuncan Davidson’s blogand is being republished here with the author’s permission.

Decision records give coding agents durable project context—as long as they don’t turn every decision into a courtroom transcript.

Architectural Decision Records (ADRs) help human teams establish rules and carry context forward in software projects. They capture significant design choices, their context, and the reasons behind them. Like many tools built for human software teams, ADRs work well for coding agents too.

Agents often arrive with little memory of yesterday and only a narrow view of a codebase. Even systems with persistent memory may preserve context without establishing whether it is accurate, current, or accepted by the human team. Decision records help them understand the intent behind the code rather than having to infer it. Keeping them in a project repository spares agents from having to trawl through issues, search chats, and perform code archaeology. When you record intent explicitly, an agent is less likely to mistake an implementation detail for a foundational rule.

Once a decision enters an agent’s context window, the agent may adhere to it even more rigidly than a human would. In my own work, I’ve seen agents fight tooth and nail to apply an accepted decision even when it is obsolete. In one case, an agent preserved an outdated storage abstraction across a new feature because an ADR still described it as mandatory. Instead of flagging the mismatch, it added another layer to keep the new requirement technically compatible with the old ruling.

The first remedy is to give agents explicit permission to question decisions that no longer fit—and to watch for signs that they’re overfitting. But that solves only half the problem. When you invite an agent to update a decision, a second tendency appears: preserving the deliberation. Every clarification becomes an amendment explaining its own existence at the expense of clarity. Small implementation details become rules, and cross-references acquire their own restatements and justifications. The result is overlitigated prose that is hard for humans to read.

ADRs should absolutely be readable by humans, especially as we lean on agents to generate more and more code. To counter this, I’ve become explicit in my projects’ `AGENTS.md` files about how agents should apply and maintain ADRs. Here’s an excerpt from one:

`Architectural Decision Records (ADRs) are stored as Markdown files in the ``docs/decisions directory. Treat accepted ADRs as binding. Proposed ADRs ``are non-binding context. Superseded ADRs are historical context and do ``not govern current work. If a given task conflicts with an accepted ADR, ``stop and discuss whether the task or ADR should change and propose the ``change that you think should be made. Propose new ADRs or updates to ``existing ones when a change introduces or revises a durable product or ``architectural decision.`

`Keep ADRs succinct. Each ADR carries only its current text; Git history ``is its changelog, so do not add or maintain amendment logs in ADR ``headers. When substantively changing an accepted ADR, add or update a ``single Updated: date line after Date:—its presence signals that history ``exists and Git has the details. A superseded ADR records a Superseded-On: ``date instead of Updated: , matching the Supersedes: line on the ADR that ``replaced it. State each rule once in the ADR that owns it and cross-reference ``it from other ADRs instead of restating it.`

These instructions are still evolving in my projects, and different projects will need different conventions. Some teams will prefer immutable ADRs that are superseded rather than revised; in my projects, I’m happy to have Git carry that history.

If you do something similar, adapt the guidance to your own needs. The essential principle is that each governing ADR should describe the decision currently in force, with enough rationale to apply it. An agent doesn’t need the transcript of every argument. It needs the ruling that governs today and clear permission to stop when the ruling no longer fits.
