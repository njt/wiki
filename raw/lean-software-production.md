---
url: https://mattwynne.net/lean-software-production
date_fetched: 2026-07-05
backfilled: true
---

## Lean Software Production

As AI accelerates software development, I see something emerging that I think needs a name. Excuse me for getting fancy, but I propose we call it **Lean Software Production**.

Or maybe Lean Software *Manufacturing*? *🤷🏼*

Lean Software Production acknowledges that software is no longer a craft, and that mass production is too rigid, inflexible, and inhumane. It combines:

- **Lean:**Industry-standard in modern manufacturing, lean thinking means managing the production system using human-centric continuous improvement, systems thinking, pull-based flow approaches.
- **Software:**The robust software engineering disciplines of Extreme Programming (XP) that keep the software malleable.
- **Production:**Agentic orchestration or dark factory patterns, where LLMs generate large amounts of code unattended.

Let’s dig into these three elements to understand why they matter and how they fit together.

## Why Lean?

When comparing the way the Americans approached manufacturing in the 1960s compared to the Japanese lean pioneers, W. Edwards Demming said:

Let’s make toast: I’ll burn it, you scrape it


Many people are working with LLMs like this today: letting the generate code and then trying to inspect the quality in through code review. By adopting the mindset of *kaizen* (continuous improvement) and practicing defect prevention, we can learn from the system’s mistakes and improve it.

Lean thinking revolutionized the manufacturing of physical goods in the 1960s and 70s, and was fundamental in inspiring the Agile Software development movement in the early 2000s. The Toyota Production System views the production line as a socio-technical system, an elaborate dance between humans and machines.

As machines begin to play an ever-increasing role in the production of software, the pace of delivery accelerates, and the role of humans in this process changes rapidly, I think these ideas are crucial for us to re-visit.

Some ideas and tools from lean, such as Kanban boards, have become a ubiquitous part of the software industry culture. Kanban is rooted in the idea of *just-in-time (JIT)* production or pull-based flow, popularized as The Theory of Constraints in Eli Goldratt’s The Goal. This idea is also the basis of a Minimum Viable Product (MVP) from Eric Ries’ The Lean Startup.

But there’s so much more in lean that we’ve yet to really embrace. I believe it can really help us in this moment.

In Lean Software Production, instead of spending time crafting every line of code, we focus our attention on crafting the system we use to produce reliable, well-engineered software at scale. By working *with* the system, continuously improving it, we’re following the lean discipline of *Jidoka:* building quality in.

What does this look like in practice? For example:

- if you notice your agent is making mistakes and generating code that you have to correct, don’t blame the model: Instead, look to how you could have improved the context you gave the model so it would make a better decision next time. Try capturing architectural decision records (ADRs) or design heuristics in your git repo where the agents can study them before writing code.
- if you’re confounded by walls of text or code coming out of these models, try asking them to build you a slide deck, or write an HTML file summarizing the key points that you need to understand and consider. Have the agent present or receive the information in a medium that works for you.

This culture of continuous improvement and just-in-time work has created great manufacturing systems, and I believe it’s key to creating great software production systems too.

## Why Extreme Programming?

When a single engineer can generate the volume of output that once required a team, the costs of problem ambiguity, architectural weakness, slow or unreliable tests and defects don’t just cause friction, they set things on fire.

In this light, the robust software engineering practices of XP finally start to look like standard industrial safety equipment:

- **Pervasive automated testing**, like double-entry bookkeeping for your code
- **Continuous integration**, so you get fast feedback about whether your changes integrate with everyone else’s
- **Relentless refactoring**and a commitment to high-quality code, keeping the software soft and malleable.
- **Pairing**on decision-making, so that the team keeps a shared mental model of where the system is going and why, through conversations.

If you’re fearful of LLMs producing slop, or building a system that nobody understands, these are the techniques that the best software engineering teams have used for decades to mitigate that risk with humans.

Now, two things have changed: you need these techniques more than ever, and it’s cheaper than it’s ever been to implement them. There’s really no excuse.

For example:

- Ask a robot to introduce mutation tests into your CI pipeline.
- Build a review/fix loop that takes on Sandi Metz or Martin Fowler’s persona, identifies flaws in your code and fixes them, autonomously
- Leave an agent in a loop reviewing your test suite for slow tests and optimizing them

To be clear, I’m not just talking about getting your agent to use test-driven development while it generates code for you. I think that problem has largely been solved. I’m talking about taking the attitude, the values of extreme programming and applying them to this agentic moment. I’ve already written about how pair programming is changing, and I recognize

Instead of cramming in more features, spend the extra time you’re gifted by these LLMs to improve your working environment, to improve the flow of your software factory.

## Towards the Lean Software Factory

Earlier this year I was fortunate enough to spend a week in a workshop with Justin McCarthy, co-founder and former CTO of StrongDM, who pioneered the idea of a dark factory for software development.

Justin’s team set themselves a rule:

Code must not be written by humans

Code must not be reviewed by humans


This sounds scary at first, but like many challenging ideas, it forces you to really think about what would need to be true for you to do it.

If you can’t review the code, how do you know whether it’s good? You have to build yourself systems that can judge that goodness for you, and you have to be able to tell them what good looks like. Good in all its forms: architecturally sound, secure, performant, user experience, to name just a few.

I think many people trying to adopt AI coding tools right now are stuck in a kind of half-way house: they use LLMs to generate code, but they don’t think about how else they can use it to modernize and automate their entire software delivery pipeline. In that way they just end up making things worse.

Annie Vella uses “middle loop” to describe a layer of work where engineers supervise AI doing what they used to do by hand. Her research shows that a lot of folks are getting mixed results with this right now, and I think that’s understandable: there just isn’t a lot of good literature or patterns in our industry for how to do it well. Birgitta Böckeler has written about harness engineering, where engineers focus less on engineering the code itself, and more on engineering the system that produces the code.

Once you’ve experienced “hands off” software development, where you neither write nor read the code, there’s no going back. The pace that you can solve real problems for people is exhilarating.

As the models get better and better, the question increasingly becomes: **what  can’t the agents do?** By asking ourselves this continuously, and using their power to automate away tasks that are mundane and error-prone, we elevate our own work to something I consider to be much more interesting: deciding what problems to solve, and how to judge that they were solved to our satisfaction.

The product is still working software, but now the work is engineering the system that produces it.

—

*If this sounds interesting to you, I am teaching a public course starting July 16th: Build a Software Factory: Hands-off agentic coding for experienced engineers.*

*This blog post was entirely organically written by my human hands and brain. I want to thank several people who gave feedback on earlier iterations: Jeremy Lightsmith, Rob Bowley, Chris Parsons, Emily Bache, Dave Farley.*
