---
url: https://threedots.tech/post/ddd-and-ai-coding/
date_fetched: 2026-09-04
---

# Domain-Driven Design matters more when AI writes your code

Whatever you think of AI coding, the way we build software is changing.
Everyone wonders, “What will stay relevant?”
All we have is opinions, so here’s mine:
**most ideas behind Domain-Driven Design are now more relevant than ever, as DDD has never been strictly about code.**
As we do more coding with agents, we still need to understand the domain, model it well, and work as a team.
Let’s see what DDD can still teach us and what AI can’t replace.

The usual disclaimer: I write about complex projects maintained over months or years, where teams tend to struggle and where DDD shines. There’s no reason to use advanced patterns in pet projects or CRUDs.

## What hasn’t changed since 2003

Eric Evans published *Domain-Driven Design* more than two decades ago, but it’s still surprisingly fresh.
The book covers many topics, but the main message is that **the hard part of software engineering is understanding the problem domain and modeling it well in code.**
It calls on engineers not to get lost in the technical details and to focus on what they’re solving,
because it’s more difficult to get that part right.

Instead, the technical talent goes to work on elaborate frameworks, trying to solve domain problems with technology. Learning about and modeling the domain is left to others. Complexity in the heart of software has to be tackled head-on. To do otherwise is to risk irrelevance.


We now see that the implementation details become less relevant thanks to AI tools. To a degree, you can be productive without a deep knowledge of a programming language or a framework, and it’s easier to move to another tech stack. If you’ve tried using the top coding agents, you’ve probably seen them quickly generate decent code.

But knowing high-level software concepts is still essential to guide agents and to decide whether the output makes sense.
Most of all, **the domain complexity (the business problem you’re solving) is still the hard part**.
The good news is that coding is easier than ever, so we can focus on these complex problems instead of getting lost in the implementation details.

Except, instead of obsessing over building the frameworks (or adding more microservices, or whatever else is a cool trend),
we now follow the model benchmarks and optimize our agentic setup to generate *better code* with less effort.
As Evans noted back in 2003, **we still try to solve the domain problems with technology.**
Funnily enough, this time the CEOs also believe it and push us toward it.

There’s just one small thing left to figure out: *how the thing should work*.
If someone tells us, we can throw tokens at the implementation and call it a day.
And if no one knows, well, maybe an agent could write the plan for us?

## The domain model isn’t an artifact

The foundation of DDD is model-driven design: distilling how the domain works into a model that’s expressed in code. The domain model is an abstract concept, and there’s no one way to represent it. You can use documents, diagrams, and code, but they are all simplifications of how the domain works. An accurate model comes from switching between design and implementation, and applying what you’ve learned over and over.

DDD calls this *knowledge crunching*.
You need domain experts (the people who know how the business works), and engineers who are experts at building software.
They work together to understand what the software should do and how to implement it.
(For example, in an Event Storming session, a workshop where developers and stakeholders map out how the business works with sticky notes.)

Figuring out the domain is a team effort and hard work, so it’s tempting to have someone else do it. For example, AI agents are brilliant at research and analysis, and can access your documents, chats, and code. It seems they could create the domain model for you, and then also implement it.

But this naive approach misses the point.
**The value of working on the model is that you (and your team)  understand how the domain works**.
An AI-generated wall of text doesn’t help you figure out the business problem.
The agent will just create an impressive artifact no one reads.

The challenge has never been to write a long document on how something could possibly work. More often, software projects fail because of other issues:

- Engineers work on the wrong thing.
- No one knows what needs to be done in the first place.
- The team tries to build everything at once (scope creep).

Here’s one trap I’ve fallen into many times. In engineering teams, it’s easy to feel like we’ve got it all figured out: we’re programming experts, know our patterns, and follow best practices. We just need someone (a business stakeholder or a product manager) to tell us what to do.

But often, the domain experts don’t have a clear plan either. They know the problem they have, but not what the solution is yet. There’s no complete document with all requirements. A developer’s first instinct may be to think, “Let me know once you come up with something, I’ll be around.” In other words, we’re here for the fun part of coding. (Note this is the part AI agents automate.)

At the other extreme, non-technical managers try to come up with the entire solution and hand it to the team to implement. Or, the most recent trend, someone uses AI to generate a full feature or product spec, doesn’t even read it, and passes it along. So the plan sort of exists, but it’s nowhere near something ready for production. (If someone claims otherwise, check how complex the thing they’re shipping really is.)

In all of these scenarios, no one has a clear idea what they’re doing. No matter who writes the code or how fast, it has little value at this point.

In contrast, when you work on the model together, your team understands the problem domain and agrees on a solution. Developers implement it, and domain experts can jump in once something unexpected comes up or another iteration starts.

Even if you do most coding with AI, **you need to get deeply familiar with the domain to guide the agent, just as you would a teammate.**
How you configure the agents and what LLM you use don’t matter as much as a clear mental model of the solution.
If I had to choose, I’d rather never write any code myself than skip thinking about what needs to be done.

## Design before ~~writing~~ generating code

Writing code has always been easier than reading it. It’s just more extreme now that an agent can single-shot a big feature for you. Even if it works, you have no idea what’s going on under the hood.

We now get stuck on code review more often, but it’s also not a new problem. It even feels familiar, like working with a lone-wolf developer who prepares complete features and drops a massive PR on you. You know nothing about the design and there are often gaps that no one has considered. Reviewing the PR is painful.

There’s a way to avoid situations like this, and I’ve seen it work in many projects.
**You design and discuss the solution as a team before sitting down to code.**
First, the domain parts: what you’re solving and how it’s supposed to work (see knowledge crunching above).
Then, the technical side.
After you kick off the implementation, there’s little chance of anything unexpected coming up.

You don’t need to agree on every tiny detail, and don’t need a complete written spec. It’s enough to sketch a high-level plan that everyone is aware of. Some sticky notes and a whiteboard are good enough.

After sessions like this, reviewing PRs is smooth. It’s easy to split the work into small chunks if you don’t discuss the entire design in PR comments. And there are fewer gaps once the feature lands in production, because you’ve already agreed on the details and everyone understands what they’re building.

Skipping the design may seem like saving time.
No one likes more meetings.
But if your team first learns about the feature from the PR, it takes much longer to review, discuss, and fix all the issues.
Since reviews are usually asynchronous, you end up with many rounds of comments.
**Code review should be a double-check that the implementation is correct, not the start of a discussion about whether the approach makes sense at all.**

**Working on design before code will also help you work better with AI agents.**
If you give the agent more high-quality context, its output will be more accurate and the coding part will be shorter.
And the more your team knows about the solution, the easier the review, even if you generate all the code.

## Ubiquitous Language: Speaking the same language as your agents

Recently, I’ve been trying to cut our cloud costs. It’s the perfect task for AI, since there’s an obvious metric to measure. I told the agent to find where our Go service uses the most memory and to cut it.

After a couple of runs, I saved us many megabytes of RAM. But then I realized memory is pretty cheap, and the updates reduced our costs by $1/month. Not surprising, right? I didn’t explain to the agent that I was trying to cut costs. I just gave it the memory usage as the target.

It should sound familiar, because it’s exactly what happens between humans in teams. Good practices that help you communicate in a team are also relevant when you work with AI. (Who would have thought soft skills may help you deal with computers?)

DDD addresses this with a few ideas around the language you use.

The first is Ubiquitous Language: speaking the same language across teams and in code, so everyone knows what you’re talking about.

In many teams, messy names are the default. In code, engineers use technical terms that drift away from the names that the rest of the company uses. Even if you feed AI documents and decision records, the agent needs to make sense of whatever language you use there.

During knowledge crunching, consider what you call the concepts in the domain. Initially, it’s common to have more than one name for each. Choose a name you already use, or come up with a more accurate one. Then, use it in the documents, spoken language, and code.

You can’t choose the names just once, because the language will keep evolving with the project. It’s something you need to care about during the design sessions and discussions.

Another idea directly connected to the Ubiquitous Language is that in bigger systems, the names you choose won’t be universal in all areas.
DDD calls each of these areas a Bounded Context.
You can think of your customer as a *profile* in the support context, and as a *user* in the e-commerce context.
You want to use consistent names within each context, not force them across the entire project.

This is especially important with bigger products, when you need to split the software between teams. If you focus too much on creating one model for all kinds of tasks, it becomes difficult to draw boundaries. With bounded contexts in mind, it’s easier to end up with a decoupled system.

Agents have access to your whole repository, and they’ll look for the names you give them. They may naively try to unify similar entities, so make it clear they are separate for a reason.

If you use well-known, precise concepts, it’s easier to express what you mean. It’ll work both with your teammates and in your prompts.

A vague prompt can make the agent spend a lot of effort on the wrong solution:

Add user to CRM and support after it’s created


You’ll make the goal more obvious by sticking to the precise names:

Once the user signs up on the website, asynchronously create: 1) a customer entry in the CRM, 2) a profile in the support system


You want the details about your domain language in your agents’ context. It’s yet another reason to keep the documentation up to date. Markdown files close to the code are a good starting point, since you don’t need a custom integration to fetch the documents.

## Developers become domain experts

The models and agents keep getting better, but you can’t fully trust them. They still make mistakes, and you need someone accountable in the end. How can you know if the outcome is correct?

**Working in the domain is how you build trust that you know what you’re doing.**
People who design the system and understand how it works can tell you whether something is likely to go wrong.

Developers become experts in the subject they’re working on. They learn skills beyond writing code, and for a curious engineer, even a complex business domain becomes intuitive with time. If you need help in that area, you know who to ask. You save yourself massive headaches by working with someone you can trust to get the job done without extremely detailed instructions.

There are two recent AI narratives. One is that anyone can now generate a clone of a well-known app. This was shocking at first, but isn’t that interesting in the long run. We already have too many half-baked apps out there even without AI.

The more interesting story is how experienced engineers can build more complex applications now. For example, a team can replace expensive third-party software with an in-house solution. But how can they trust their version works well?

**Because of the knowledge they already have thanks to working in that specific domain.**
They combine what they know with AI and build their own version so fast, it’s cheaper than paying for the license.
On top of that, they can also add any custom features they need.

For employers, having product-minded engineers around who know the domain well is huge. They can figure out the obscure details and guide the discussions. It’s always been naive to treat developers as people who code what you tell them, and now it’s even more so.

For engineers, learning to work with an unknown domain pays off more than focusing on technical skills alone. It’s easier to work in a profit center if you’re good at solving business problems, rather than being an expert in one framework.

## Don’t delegate thinking

DDD shows how there’s much more to software than writing code. If you’ve worked with complex systems, I’m sure you’ve seen it many times.

This topic brings back memories from my first software job.
I once worked on a tough problem, and spent hours doodling with pen and paper.
I finally figured it out, even though I barely touched the keyboard.
I had only recently started, so I was speechless: “Wow, today I got paid to *think*.”

This felt wild at the time, but today it feels even wilder. Back then, my manager wouldn’t blink an eye if I said I spent a day thinking about a problem. Today, we focus more on the output. Why not have the AI do the thinking instead?

For one, because the only reason I can judge whether the agent’s output makes sense is that I’ve spent long hours thinking about such problems in the past.

There’s the argument that you no longer need this experience, since AI will handle everything for you anyway. Relying on AI could eventually become the new normal, but there’s one issue I can’t ignore. You need to exercise your brain to work on complex problems, and you can’t build new mental models without thinking.

The best analogy I have is reading a book versus a summary of it.
Reading forces you to think about the topic for hours.
You slowly build a complex model in your mind, and you connect it to other things you know.
**You understand the topic, instead of reading a few sentences that you forget ten seconds later when you close the browser tab.**

No one knows the best practices yet, so it’s a good time to experiment and see what works. For now, I’ll mix old-school problem-solving with trying out new ways of working with code.

In this post I focused more on the philosophy behind DDD than individual patterns. I plan to write more, including about the tactical patterns closer to the code. I still find them useful, even if I don’t type them by hand. Sign up below so you don’t miss the next posts.
