---
url: https://mattwynne.net/dont-fear-the-dark-factory
date_fetched: 2026-07-05
backfilled: true
---

## Don't fear the Dark Factory

A few months ago our boss challenged us to adopt the Dark Factory pattern for agentic software development. Inspired by the work of Justin McCarthy at StrongDM, where they committed to producing software where humans neither read or wrote the code, we started to explore this exciting and daunting technique.

I'm someone who's always taken pride and enjoyment in crafting solutions in software. I remember reading Mary and Tom Popendick's book, Lean Software Development, where they talked about conceptual integrity and how the internal design quality of a piece of software is reflected in its usability and ultimately its value to its users. If the code is shit, that leaks out. You can feel it when you use it.

When I first started at Mechanical Orchard, in February 2024, I'd barely even used AI. At that time the best you could do was copy and paste a snippet of code into a ChatGPT window and roll the dice to see whether what you got back was just a hallucination, or something useful. At the time I was pretty sceptical and just kind of buried my head in the sand and hoped that it would all go away.

A year or so later though, it was clear that it wasn't going anywhere. I started to turn towards this technology, really try to understand it and figure out how to make it useful to me.

As the models got better and the tools like Claude Code came out, my confidence in using them increased, and I really started to enjoy it. I played with Ralph Loops and eventually built a tool for myself using a language I'd never read or written before. I’ve still never read the code, but I use that tool every day.

But how can non-deterministic coding agents possibly be trusted to produce entire systems where nobody has read the code? Won’t it be garbage? I think a lot of my friends in the agile/XP community still feel like this, and I understand why. Friends, this post is for you!

Let’s get into what a dark software factory actually is, and why you shouldn’t be afraid of it.

At its heart, any dark software factory is just a really simple loop.

Each of the nodes is an agent session. By putting them into a loop like this, with a well-designed validation harness, and a good quality seed as input, you facilitate these non-deterministic agents to naturally converge on the solution you want.

Now it's important to consider scale here. We're not talking about a factory where metal comes in and cars come out. Not necessarily anyway. We're talking about automating mundane, repetitive processes that would normally need a human in the loop, but where the desired outcomes can be judged by clear heuristics.

The better and braver you get at this, the more ambitious you can be about the scale of those processes, or composing them together. But it's perfectly fine to start with something small and boring.

For example, on the yaks project, I have a series of ADRs, architectural decision records, that describe the architectural structure I want the application to have. Periodically, I run an architectural review where I ask an agent to compare the actual code with those ADRs and notice where they are incoherent. It comes back with a list of recommendations, and we pass that list of recommendations to another agent to implement the first one. We run our automated tests of course, but we’re not valid until all those recommendations have been addressed. So we loop.

I can leave this thing grinding for an hour or more and when I come back the integrity of the code has been improved according to my design heuristics, with me barely having to lift a finger.

So we don't just have to use dark factories to generate even more implementation code. We can use them to perform maintenance tasks that actually improve the quality and integrity of the code we're writing, provided that we know how to provide that harness that will guide the agents towards what good looks like.

You can still write the production code by hand, if you like! Then have a factory grind on mitigating the security vulnerabilities, or merging dependency upgrades, or running and triaging mutation tests.

Of course you absolutely can have a factory that writes production features for you too, but that’s only one way of using this pattern.

This is where the whole thing starts to remind me of learning test-driven development. When you first start to learn TDD, it's hard when you're used to starting from the implementation to have to think about where you want to go before you go there. It's hard to describe in a test what you want because you have to think about what you want without having had the chance to explore the path towards it.

In the same way, designing a dark factory is challenging because you have to create this validation harness, and that forces you to think about what you want, before you have it. This still feels a lot like TDD to me, just on a bigger scale.

*No tokens were spilled in the writing of this post. This is entirely hand-crafted, artisanal writing.*
