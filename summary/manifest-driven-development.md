---
url: https://spacedock.md/blog/manifest-driven-development/
title: Manifest-Driven Development
author: CL Kao (Chia-liang Kao)
date_fetched: 2026-07-05
date_published: 2026-06-25
---

In my 35 years of tinkering with software, coding agents have made every other technical breakthrough look like an ergonomics improvement or an evolution. Those were great, but they no longer seem revolutionary now. I think we need to talk about a new emerging paradigm. This is not just vibe-coding, agentic-engineering, or the move from test-driven to spec-driven. Those augmented our current way of working, to the extent that one can build what used to take a team. It's a hyper-multiplier for those with experience in software engineering and/or running software teams. But I think there's a new paradigm that is fundamentally different. I call it manifest-driven development for now, and it is most obvious when building out agentic systems with the system itself.

Manifest-driven development TL;DR: we can ask coding agents to assume some functionality exists already, and gradually materialize the deterministic parts and fix the quirks along the way.

Here's an example of me building out the Spacedock support of the Pi coding agent, on Pi, before this feature is implemented:

I launched the Pi coding agent with the Spacedock skills and said: "Spacedock doesn't work natively on Pi yet. But let's pretend it does and assume the behavior for the entire session, and file all the quirks we run into as we work through this."

It tried to boot up the system and realized the missing subagent integration. That was resolved, and it worked out 15 other issues to be filed. It told me I could now launch Spacedock on Pi natively, and continued ironing out those remaining issues.

We launched a "sprint" with Pi on Spacedock natively, plus the handoff prompt to fix the remaining issues. It first spiked to test the underlying capability of Pi, and then built out the rest of the missing functionality and added CI.

That felt super weird. The most similar realization I had was when git chose its content-addressable storage backend. "Oh, this means we have a way to point to all the code that can potentially be written (and maybe it already is, in a parallel universe, and just needs someone in this universe to manifest it)." But the system can now actually "fake it until it works."

Where we are going

Test-driven and spec-driven are means to the end of describing what we want at different levels of detail. Manifest-driven development is like spec'ing out in a few iterations of actual usage. And when a system can be aware of its functional limit, it can alter itself according to the users' intents and inferred desired behavior.

Why do we need to be stuck with really bad software?

We are still missing a few key pieces here:

Durable spec format: a consistent behavioral spec, a functional spec of the desired state of the product, so the agentic system knows what to assume.
Agentic-native workflow engine for encoding user intents and the autonomy levels that users want to operate in.
Rich UI representation that is fast enough for just-in-time "fake it until it works" in a rich and malleable interface suitable for human/agent interaction.
This is no longer no-code or low-code. This is mostly no-code, and just-in-time code when needed.

When I first created spacedock.md, it was mostly for automating myself out of being the human connector between agents, while still retaining controls for dialing how much I trusted the system. But this now feels a lot bigger.

If you are interested in this direction of the industry, follow spacedock.md for the development, and try out manifest-driven development in your own projects and share how it felt!
