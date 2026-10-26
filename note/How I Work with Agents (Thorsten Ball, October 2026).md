# How I Work with Agents (Thorsten Ball, October 2026)

Thorsten Ball's thirteen working habits for agent-driven development, as of October 2026: frontier models only, prompt the What not the How, make the whole SDLC legible to the agent, ask for irrefutable proof, go async, and review only high-blast-radius code. It reads as a mature practitioner's answer to the question the [[agent-coding-workflow]] topic keeps circling: what does the loop look like once you stop supervising every keystroke?

---

## What it says

- **Latest models only.** "If a model is wrong 10% of the time, you don't know which 10%, so you end up making multiple attempts." A cheaper-but-dumber model is a false economy.
- **Don't fear slop.** The "unmaintainable mess" worry "maybe was true six months ago."
- **Prompt the What, not the How.** "Make the bytecode compiler and VM AS FAST AS YOU CAN. Do not stop until it's 2x faster."
- **Make the SDLC legible.** "If the agent can't see something, it makes it up." Let it run what it builds, read production logs, deploy, flip flags, triage bug reports.
- **Eval agentic knowledge.** AGENTS.md, skills and docs are only real if agents actually use them — spawn blind control-vs-treatment agents to find out.
- **Delegate iteration.** "Generate 20 variations and a little page that lets me combine and test them" — and for problem space: "Give me 4 theories on why this is slow, then check all four."
- **Irrefutable proof.** His "biggest game changer": end-to-end walkthroughs, demo videos, before/after numbers. Put "this is not evidence" in your AGENTS.md.
- **Async over TTFT.** "Let them wait on you." Batch instead of back-and-forth; use schedules and heartbeats ("Check every day at 9am").
- **Stop managing context by hand.** "One thread = one task. Some threads are 46 messages, some are 3,714."
- **Threads are company memory.** In Amp, threads link to git commits, so "why did we do this?" has an answer.
- **Review by blast radius.** Only <5 central files and database migrations. The bar: does it work, do I understand it at the arrows-in-a-diagram level, did the agent test it end to end, can I change it in a month?

## Key quotes

> "If the agent can't see something, it makes it up."

This is the load-bearing claim of the whole thread, and it's the same insight that drove tool calling in the first place — hallucination fell when agents could touch reality. Ball's extension is radical: legibility shouldn't stop at the dev loop. If an agent can deploy and read production logs, the human's job shifts from executor to auditor of evidence.

> "Agents are really, really good at producing proof… Put 'this is not evidence' in your AGENTS.md."

The two halves of this quote are in productive tension. Agents generate impressive-looking evidence cheaply, which is exactly why you need a standing instruction that flattery-shaped artifacts don't count. Trust here is calibrated, not granted.

> "I don't care anymore. Does it work? Do I understand it at the arrows-in-a-diagram level? Did the agent test it end to end? Can I change it in a month?"

An honest, slightly chilling review policy. It trades local readability (names, duplication, style) for global maintainability questions. That's defensible — but note it assumes the codebase is regenerable on demand. Where code is load-bearing and slow to rewrite, this bar is thinner than most teams can afford.

## Themes

#concept #person #pattern

## Analysis

The thread's most interesting property is what it *deletes* from the craft: manual context management, most code review, synchronous iteration, and waiting. Ball is describing a supervision regime where the human's remaining leverage is concentrated in three places — choosing what to ask for, verifying proof, and curating shared memory (threads + AGENTS.md + skills).

It's also candid about unresolved problems: local dev with many agents is "yucky" because of resource conflicts, which is why remote machines win. And "agents shouldn't write for humans" is a sharper line than most take — it accepts that agent prose is for agents and machines, with a short exception list.

The sceptical reading: this is a statement of one person's tolerance for risk at a company (Amp/Sourcegraph) with unusual tooling — custom plugins, thread-as-memory, heartbeat scheduling. Transplanting it into a shop with slower deploy pipelines or compliance review will not go as smoothly. But as a directional marker of where practitioner norms are heading in late 2026, it's one of the clearest statements available.

## Related pages

- [[How I Actually Use Agents]] — a companion practitioner log; Ball's thread is more compressed and more radical about review and context.
- [[How I Prompt (Thorsten Ball)]] — Ball's earlier prompting habits; "prompt the What" extends that thread's thesis to whole tasks.
- [[Don't Wait for Claude]] — makes the same async argument; Ball's "let them wait on you" is the crispest phrasing of it.
- [[We Have Proof Automation Now]] — the case that verification is the new bottleneck; Ball's "irrefutable proof" habit is the individual practice matching that thesis.

---
*Sources: [[raw/2114386050638336082]], [[summary/2114386050638336082]]*
*Last updated: 2026-10-26*
