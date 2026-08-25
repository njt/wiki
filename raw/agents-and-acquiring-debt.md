---
url: https://blog.bl00cyb.org/2026/07/agents-and-acquiring-debt/
date_fetched: 2026-08-25
---

I had a lovely chat with Kate Chapman and Tom Henderson recently, where we discussed how the shape of technical debt is changing as people become “genie wranglers” and “robot life coaches” (their clever terms, not mine). Specifically, what does using AI mean about addressing pre-AI and accruing post-AI technical debt. Our main takeaway from this session is that commitments should happen at “the last responsible moment,” and AI pushes that moment later.

**Human context: ***Cowritten with **Tom** and some AI agents; lovingly curated, proofread, and fact-checked by us humans, including **Mark**. We’re still exploring these ideas, so very much consider this open for feedback. We tried to bridge our academic approach with making things useful in a business sense. We are assuming you have made it past the “**spicy autocomplete**” phase of using LLMs in your work for this piece.* *It’s up to you how much you care about code quality for the purposes of this piece, but we think we’ll get into that in a potential later piece. In this article/series, we assume debt is taken on intentionally. This is often not the case, but that is a different problem space than the one we intend to explore.*

AI technical debt is different from software debt because it shifts when commitments can be made in the arc of gathering information. We have historically taken on debt early in order to realize some value sooner. Now, we can gather information more cheaply earlier (in theory), and make our difficult-to-change decisions later. That changes the type of debt we’re taking on. It also makes testing even more necessary. You won’t be surprised to know that we’re highlighting and contributing to human judgement throughout, rather than trying to replace the humans. We hint a bit at product commitment points-in-lifecycles and decision making throughout this post, but we said NO to that side quest (for now).

**Debt is an instrument, not a smell**

Cunningham’s actual meaning: shipping expedient-with-knowledge-of-cost to get customer value sooner. EG, deliberately selecting a less robust and “we know we must replace this” design for now in exchange for speed to get in front of a customer. This meant investing lots of time in making decisions early, before putting in a great deal of work, in order to make sure the work was the right work for this piece right now, on the right foundations, for potential future work.

The introduction of AI allows you to move faster in all sorts of ways. But which ways are useful for system goals? We think that if you change consideration of AI from “build this for me, faster” to “when do decisions need to happen?” you can shift commitments to later in the process, which shifts where the debt happens and what sort it is.

Traditionally, we’ve needed to invest a timeboxed but in-depth amount of work into foundational commitments in order to make the “less wrong” commitment that brought value soon enough for business needs. Now that code is “cheap,” we can make decisions about what the right work is differently, often by trying out different options before deciding which is best for our use case. The work to understand the right decision becomes understanding what experiments to run and how to choose based on the results, and then building the more correct robust path later than you would have otherwise started.

**Agents acquiring debt**

Because debt is “gain value now, pay later” and agents let you take it on faster and more deliberately, more shipped experiments per unit time shift where the debt is. At its more basic, generating inside a bounded context window produces misalignment and duplication debt by construction. This is similar enough to traditional technical debt that we don’t worry about it much in this supposed series beyond testing considerations and tooling improvements.

There is something truly new here, however. Classic debt was at best decided as “expedient rather than right, knowingly”; agent-generated debt is currently equally accepted-without-agreement, but now is not even attempted to be surfaced for awareness in the first place, and is generated at a much higher rate. This generates “comprehension debt,” and it has a higher interest rate than the technical debt.

Because it is easier to take on debt now, and theoretically easier to address it later, it is even more important to be deliberate about this, especially where shared infrastructure comes into play. Mark offers this metaphor: “AI companies that sell coding agents are the equivalent of predatory credit card companies offering introductory cards on college campuses to freshmen.”

**Comprehension debt**

This is the AI-native debt type, and is the new highest-interest loan you can take out against your code. Accrued most obviously when we “LGTM” an agent’s command or edit, it can be paid down when we ask them plain-language questions about what the system does, or what this commit was about. They answer *what*, not *why*: they have the current state and the commits, not the folklore or the rejected alternative with reasoning. There’s a symmetry here worth making explicit: human folklore was always imperfect, too (EG the person who knew why this is the way it is has left). 

Agents didn’t create the why-problem; they universalized it. Every agent is the new hire on day one, forever. This might have the same fix as always: record decisions when you make them (with, say, ADRs), except now agents can help write the ADR at decision time, and agents are also its hungriest readers. This of course has some implication of having agents throw each other under the bus when you are hoping to use them to close on comprehension debt. Cue dueling sycophancy banjos about whether you or other AI is more important to pander to.

**Agents and paying down debt**

Mechanical transformations are “cheap” now, and refactors that were considered unaffordable seem affordable. But also, cheap-looking paydown churns the foundation others are building on, competes for tokens and attention, and itself generates comprehension debt. Whether or not it’s worth it will be based on how good your definition of “good” is for the refactors, what else you have going on, and what depends on a predictable code base (more in the future on determinism, probably). Automations that were unaffordable are also now affordable; This means you can do things like write very specific linters to avoid or require certain patterns (which was not something most teams could invest in before). These in combination with prototyping out various approaches more cheaply further in advance means your “last responsible moment” for product commitments shifts later, changing when you accrue your forms of debt.

**The last responsible moment**

What all this means is shifting what kind of information you get about your system to earlier in the production cycle, and experiments leading to that information is “cheap.” Making a decision later is generally smarter because Future You knows more than Present You (supposedly), always, which contributes to determining the “last responsible moment.”

Before you get all “I didn’t procrastinate, I maximized my information about the situation,” pause for a moment to consider the costs of delaying. Just as we time boxed architectural choices before needing to make a choice and accrue debt to actually ship anything ever, don’t let evaluating technical prototypes prevent you from making clear product decisions early and often. Accounting for “holding costs” in determining your “last responsible moment” will curtail sidequests and keep your project running tightly and in its desired direction.

**What else this means / future things to talk about**

This was a doozy of a post to write. Tom and I are getting to know each other’s styles, we had beaucoup side quests we wanted to go on. Where we’re at is that LLMs shift the *entire system* of how software development works. If those pieces are not moving together in an orchestrated way, it is possible to end up further behind than you would have been without involving AI in the first place. We’d like to explore what that system is, how different parts of it are changing, and what that means about what the individual pieces of work are. Unless of course our collective ADHD brains decide to fixate on something else before we’re done.
