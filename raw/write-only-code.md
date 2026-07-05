---
url: https://www.heavybit.com/library/article/write-only-code
date_fetched: 2026-07-05
backfilled: true
---

# Write-Only Code

*Everything that follows is written through the lens of enterprise software. Large, revenue-generating software systems historically built and operated by teams of hundreds or thousands of developers.It has implications for other classes of software, but that is a discussion for another day.*

Six months ago, if you had asked me how much production code would eventually be written by AI, I would have claimed a large percentage. LLMs are clearly a massive productivity boost for software developers, and the value of humans manually translating intent into lines of code is rapidly depreciating. I also believed, and still do, that humans whose primary job is to build and operate enterprise software are not going anywhere, even as their day-to-day work is fundamentally redefined by this newest abstraction.

What I underestimated was how little of that future work would involve reading code at all.

I am now convinced that for better and worse we are barreling toward a future where a large and growing fraction of production code is never read by a human. Not skimmed. Not reviewed. Not tweaked. I have taken to calling this **Write-Only Code** (shout-out to Waldemar Hummer of LocalStack for helping coin the term) and have been spending a lot of time thinking through what it means for us as an industry.

“AI writes the code” is already true inside many enterprise teams, but today it mostly means some form of AI-assisted pair programming. Humans decompose work into small tasks, hand them to agents, review the resulting pull requests, make edits, perhaps re-prompt, and iterate until the result is good enough to ship. This was a sensible practice and necessary to reliably ship production code with models that existed before late 2025. Subtly but importantly, in this workflow the requirement for human review also preserves the rest of the software development lifecycle, because human review remains the primary bottleneck through which all production changes have to pass.

That bottleneck is going away.

Recent step-function improvements in model capabilities are the first step in breaking many of the core assumptions underpinning the software development lifecycle (SDLC). These agents can now successfully handle much higher-level chunks of functionality. With emergent techniques that allow agents to plan, execute, and self-correct over long horizons, we are already seeing experiments demonstrate the shocking scope and complexity of working software they can produce. As these practices are adopted we will start producing software at a volume and pace where even if we wanted to, there will never be enough human bandwidth to review it line by line.

Unlike some AI maximalists, I still do not believe that “no one reads the code” implies “no humans involved.” Enterprises still require accountability. Someone must own the system, its outcomes, and its performance. Someone must answer for incidents, and ultimately be on the hook when things go wrong. Enterprises buying software are looking to their vendors for far more than just code.

The role of a software engineer has always been to sit between poorly specified, natural-language business goals and (mostly) deterministic machine behavior, while navigating a web of constraints. You can distill that role even further to “reducing risk.” Reduce risk that the business does not have the capabilities it needs to compete. Reduce the risk that the software is broken, that it cannot scale, or that it cannot adapt as business goals shift. That role does not disappear in a future where **Write-Only Code** flows directly from LLMs into production, but the culture, processes, and tools required for humans to fulfill it need a fundamental rethink.

**History Doesn't Repeat Itself, but It Often Rhymes**

Software development and delivery has a long history of eliminating one bottleneck, only to immediately confront the next. As we stand on the precipice of removing perhaps the biggest bottleneck ever, humans writing code, it is instructive to look at the last one we removed.

It is easy to forget that just a few decades ago the practical bottleneck for running new enterprise software in production was not writing code at all. It was hardware. Procuring servers, waiting for delivery, getting them racked, configuring networking, and then, months later, finally putting something into production. The combined emergence of continuous delivery practices and on-demand, ephemeral computing in the mid-2000s collapsed that constraint.

What followed was a new wave of tools and practices that assumed a fundamentally different posture toward production. DevOps and “pets vs. cattle” was not just a slogan, it was a rewriting of expectations. Production servers stopped being precious, named, and lovingly maintained. The idea that working servers could be stamped out programmatically, and explicitly not intended for human access, once seemed absurd. Today, if someone in a high-performing team shells into a production machine, it is considered tainted and scheduled for replacement by a pristine instance.

This shift moved the bottleneck squarely to developer velocity. Once infrastructure could be provisioned on demand and incremental changes could reach production as fast as they were created, the limiting factor became how quickly a human developer could translate business requirements into running software. This unleashed a new wave of productivity and, in my opinion, is why developers became the new kingmakers.

At the last startup I founded, this reality shaped nearly every decision we made about how we worked. I was maniacally insistent that any proposed change to our SDLC be evaluated first through the lens of developer velocity. Changes that safely improved developer velocity were effectively mandated. Changes that slowed developers down were heavily scrutinized and, if not rejected outright, at least tuned to minimize their impact. Over time, my team came to half-jokingly call this heuristic “Ruscio’s Law.”

What we treated as a simple rule of thumb is rapidly evolving from an optimization for elite teams into a prerequisite for survival.

**From Review to Trust**

For most of modern software history, human code review has served as the final backstop for confidence in production systems. Tests can be imperfect, monitoring can be incomplete, and bugs can be subtle, but in the end, skilled engineers who both create and review the code can reason about what it might do. **Write-Only Code** breaks that assumption at scale. If we are going to ship code that is never read by humans, we need other ways to gain confidence.

Much as humans no longer shell into individual production servers, I believe we will develop similar practices around unread code. Over time, we will treat “humans had to read this to be comfortable” as a smell in our code generation pipeline, or as an explicit, expensive trade-off reserved for truly mission-critical subsystems. A natural outcome of this shift is a “code reading coverage” metric, tracked much like test coverage. What fraction of production code has actually been read by humans, partly as a safety signal, and partly as a metric teams deliberately and safely work to drive downward toward an asymptote.

Pragmatic teams will not adopt **Write-Only Code** everywhere at once. They will identify where it is safe to begin and where traditional human review should remain. Understanding and controlling the *Slop Radius*, how far unintended behavior can impact before being detected or contained, will be a critical skill for teams to develop. As practices and techniques mature and the guarantees we can extract from automation improve, the surface area of unread code will expand and the scope of manual review will shrink.

The question for engineers is not whether this shift will happen, but what primitives will replace human authorship and review as the foundation of trust.

**How the Engineer’s Job Changes**

In the AI pair-programmer story, the human engineer is still primarily an author and reviewer. In the **Write-Only Code** story, that same engineer becomes a systems designer, a constraint writer, and a trade-off manager.

You spend more time shaping intent than shaping implementation. You obsess over interfaces, invariants, failure modes, and the conditions that must hold true. You decide what still requires human review and what explicitly does not. You invest in the tooling that makes “ship it blind” not a reckless act, but a competitive advantage. You reduce risk.

You also accept that the SDLC is being rewired even when the external inputs and outputs look the same. The business still asks for software and receives working software, but the inside of the factory is fundamentally different. That difference will decide who wins.

There is also a psychological shift that should not be understated. Engineers have long taken pride in crafting and deeply understanding what they ship. Entire books celebrate *Beautiful Code*. In a world of **Write-Only Code**, that pride shifts toward building systems that remain correct without requiring comprehension of every line. “I write code” becomes “I build software.” Human engineers will still be accountable for outcomes, but they will increasingly seek that confidence without ever reading the code.

**In Conclusion**

**Write-Only Code** is not a prediction about what we should want. It is a description of what happens when software production scales beyond human attention. The question is not whether humans should remain in the loop at the level of individual lines of code, but whether we are willing to take responsibility for systems whose behavior we can no longer fully inspect. We have been here before. Each time a bottleneck falls, the industry reorganizes around what replaces it.

The mistake would be to treat unread code as a failure of discipline rather than a signal that discipline itself must change. Human review does not disappear because it is unimportant, but because it no longer fits the scale and shape of the problem. The organizations that succeed in this transition will not be the ones that cling longest to familiar rituals, but the ones that invest earliest in new primitives for trust, accountability, and control. Refusing to adapt does not preserve safety. It simply ensures that adaptation happens accidentally, under pressure, and without intent.

The role of the human engineer has never been to type code for its own sake. It has been to reduce risk in the face of ambiguity, constraints, and change. That responsibility not only endures in a world of **Write-Only Code**, if anything it expands.

The next generation of software engineering excellence will be defined not by how well we review the code we ship, but by how well we design systems that remain correct, resilient, and accountable even when no human ever reads the code that runs in production.

### Content from the Library

### What if the AI Harness Was Your Computer?

Giving AI Data Ownership Back to Users Researchers have warned for years: Frontier models are potentially running out of data to...

### How to Think About Agentic Memory Job-to-Job

Moving Memory From Individual Apps to Centralized Infra Despite all the enthusiasm for AI agents, recent reports suggest that...

### Why Orchestration May Be the Future of Agentic Development

For now, AI agents are autonomous entities to which users can delegate simple tasks: Monitor your calendar. Sort emails. But for...
