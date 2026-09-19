---
url: https://blog.detail.dev/posts/towards-self-driving-codebases/
date_fetched: 2026-09-19
---

Agents can oneshot games that are actually fun. With the right guardrails, agents can execute incredibly impressive migrations in complex codebases, even rewrites in new languages. But basically all “real” software work still has human engineers driving the process. How do we get to a place where a much bigger portion of the work gets handled for us, without requiring our attention?

It seems like agents should be able to build entire software systems themselves. Why is this going so poorly in practice? What new primitives will we need to make it all work? How far can this go?

## What comes after tokenmaxxing?

A lot of engineering orgs spent the first half of this year offloading as much work as possible to armies of agents and adversarial loops. The results have been pretty disappointing: mountains of dubious code, but no tsunami of incredible software. The ROI on all those tokens has been sketchy at best.

In hype cycle terms, we’re in the trough of disillusionment. We’re working out how to deliver on the utopian vision we had a few months ago that isn’t working yet in practice.

What happens next? In every other adoption cycle, the answer has been: figure out best practices, figure out where our shiny new hammer is well-suited for a nail vs where we’ve been using it to smash screws, and then create the building blocks we didn’t realize were necessary until we got our hands on the shiny new toy.

One diagnostic we can use to help us get there is to ask: when the software mostly drives itself, what will engineers do?

A common answer is: setting up Loops! We used to write code, then we wrote prompts, now we set up loops of agents and write `/goal` a lot.

I think this is mistaken. Right now, setting up a viable software loop that produces net-positive software change without incinerating money is a ton of work, but mostly that’s because the toolchain isn’t ready. When we have the right moving pieces, these loops are going to be easy to set up, easy to trust, and cost-effective. This won’t be where our time goes.

Rather, the most valuable engineering work is going to be *having good ideas*. The important ideas are still coming from outside the software factory. In fact, the purpose of this whole machine is to minimize the amount of things we have to think about that aren’t high-upside.

- **Killer features are still company-making.**Cursor Tab, Descript’s transcript editing, OpenRouter abstracting inference vendors behind a common billing interface, these are all company-making ideas. Beyond creativity, good ideas require problem intimacy and domain expertise, and engineers will need to contribute to that process. [1]
- **Killer architecture is still high-leverage.**The right simplification still saves an enormous amount of complexity later on, which manifests in fewer bugs and easier product iteration. A little bit of forethought makes high-value features low-hanging later on. There’s still a lot of value in working out a more correct data model or adding a queue in the right place or building the scaffolding for a better way to test the product.

## What parts should be self-driving?

Outside the realm of “having good ideas”, we should be trying to move codebase work to GPUs. For example, can agents handle these whole concerns?

- **Detecting and fixing most bugs.**The new feature doesn’t work right if a user is logged in via SSO. Bulk delete doesn’t update the row counter at the top of the page the way individual delete does. An agent should catch these for us and fix them. Most bugs tend to have- *obvious intended semantics*– we don’t need a human engineer to adjudicate how the software is supposed to work, and we may not even need docs or a spec either.
- **Debugging production errors.**When an error shows up, an agent should correlate it to a recent commit or traffic change or infrastructure difference or unexpected condition and, usually, patch it for us. Backends should basically never throw 500s anymore. Browser console logs should have basically no errors.
- **Optimizing agents.**How much should we really be writing agent prompts? Can Braintrust or Raindrop or Arize or Langfuse isolate common agent pathologies, propose updates, backtest them, and ship them?
- **Frontend consistency.**The visual language of any app should be internally coherent. Apps should use consistent fonts, colors, iconography, spacing. The design system should bootstrap itself and enforce itself, with humans making confirmatory decisions.
- **Application polish.**Apps should behave in ways users expect. Ctrl-click should open a link in a new tab. Disabled buttons should have an explanation why. Form fields should be preserved on refresh. UIs should render on mobile in a reasonable way. UIs should be navigable by screen readers. There are dozens of basic quality-of-life expectations like these.
- **Growth optimization.**Iterating on form conversion, optimizing onboarding, churning out marketing pages. There are playbooks of experiments that work pretty well, and agents should run those playbooks for us. [2] This is not to imply that growth is dead – it’s more alive than ever! But the work is going to look more like schemes and heists now, not optimizing CTAs. Growth teams set goals, agents iterate not just against the code but against reality by putting changes in front of users and learning, we all get to focus on higher level ideas.

The list goes on. GitHub Copilot gave us autocomplete for lines of code, and agents will give us autocomplete for entire products.

## Missing Primitives

If this is going to work, the dev stack is going to need some new primitives. These are all going to be table stakes:

- **Agent-legible dev environments.**The bugs, overwhelmingly, come from the areas where agents can’t see them . If your app has third-party integrations that your agents can’t exercise end-to-end, those integrations will have bugs. If your repo doesn’t have a good agent-browser setup, the whole frontend is a gray zone where your agents are flying blind. And so forth.
- **Global memory.**It’s actually fine if agents make a lot of boneheaded mistakes. What’s not ok is if they keep making the same mistakes. If you have to tell an agent that the audit log table is append-only and should never be updated in-place, that’s ok, but if you have to subsequently tell every agent that touches that area, you’re back in the loop for low-level work. Whatever memory system we create needs to work across the toolchain – the code review bot needs to be aware of the correction you issued to the agent that wrote the code in this area two weeks ago. When you scold an SRE bot for doing an unsafe prod operation, the agent needs to understand why, and then every future agent needs to be aware of this correction. Global memory is mostly a question of waiting for tools to improve. Either we’ll create an open standard, or we’ll use consolidated suites that provide daily driver agents and reviewers and everything else we need all-in-one, or both.
- **Codebase rot prevention.**Are agents leaving behind dead code? Are there five ways to do something that should have one way? Is the type system intuitive? Is the data model aligned with the product we’re trying to deliver? Something needs to handle all of this, or else the codebase is going to start spiraling.

We still need most of the primitives from the last era too, e.g. APM (but it will be queried mostly by agents reading logs, not widget-rich UIs) and CI (but with better merge queues) and A/B testing (but operable by agents).

## How do you build an agent-legible dev stack?

Where engineering teams need to do a lot of work right now is in creating really good cloud dev environments that make it easy for agents to exercise the code in all the ways it needs to be exercised before it’s possible to even know if it has bugs. **At this point the limiting factor for dev agents is the environment they operate in, not the models and harnesses themselves.**

Put another way: the agents are good enough to do a lot more of the work, *if* they’re operating in good enough environments. But they create bugs where they can’t see: race conditions the agents don’t have a systematic way to reproduce, slow queries where they’re ignorant about the shape of the data. [3]

Where agents have blind spots, they make mistakes, which limits trust, which means work needs to be reviewed with more scrutiny, which limits how much we can hand off.

So the most important investment an engineering team can make right now is in making their dev environment extremely friendly to agents. But right now that’s a bottomless pit of schlep and oneoffs with no way to quantify how you’re doing. It takes good ideas and focused engineering to fix these dev environment gaps. A local dev setup with representative test data is a moving target. The right new abstractions to make it easy to write good tests are hard to come up with. And so on. This work is challenging!

This work is also codebase-specific, and it’s still an art that very few people can do well. That’s one reason so many software factories are so disappointing. We need much better ways to make this kind of investment.

That means there’s a lot of leverage in helping teams get their dev toolchains agent-ready. How can an engineering team know what the low-hanging fruit is? How should they prioritize the work to make their factory work better? How can they know if they’re doing a good job? If we can turn this into a science instead of an art, and help engineering teams make methodical progress, the factories will start humming.

Here’s our best guess as to the path forward:

- Mine codebases for bugs. Get very good at picking bugs that matter.
- Fix the bugs. Track the gaps that make it hard to validate that fixes are correct, or that bugs are real to begin with.
- Use the traces from fixing these bugs to prioritize the work we need to do to make a codebase more amenable to agents.

Put another way: Detail’s secret plan is to create a product that bootstraps its own work, and then use that work to help engineering teams know what investments to make and how well they’re working.

If we can bootstrap a corpus of useful work for any codebase – in this case, bugs the eng team would want fixed – we can use that to determine the high ROI ways to make any codebase better for agents. We can benchmark how agent-ready a repo is. Engineers can grind on the gaps that matter in their dev environments until almost all low-level bugs can be handled autonomously, and then engs can focus on the real high-value work: having good ideas for features and figuring out abstractions that will simplify implementation.

That’s how we’ll get to the “plateau of productivity” and it’s how the best engineering teams are going to be operating for the next six months. Not tokenmaxxing, certainly not banning AI, but using a process like this to hillclimb agent readiness and progressively handing off more and more of the work.

We’re launching a piece of this today, but you’ll see these ideas continue to take form in our product as we build towards this vision.

## This isn’t new

Technology advances by handing off lower level concerns to systems that manage them for us.

In the 2010s we moved our software to the cloud, and now we don’t have to think about where it runs, physically, or even which virtual instance each service running on. A smaller, more focused engineering team can build a successful software product now. We get to live in the logical plane while platforms handle the rest. But we went through a similar transition that was very messy at first, and we had to figure out a lot of missing primitives along the way.

It used to be a multi-month project to set up CI and CD, but now that we have the right primitives and mature tools it’s an afternoon of work unless you’re migrating a bank mainframe or something. An analogous transition will happen with “loops”.

We’re hiring! And if you want to come along for the ride, try it out: https://detail.dev/

[1] If “taste” was ever actually a differentiator, it isn’t now. But good ideas are.

[2] Some app domains are pretty well understood. For an eCommerce website you’re going to want cart abandonment emails, “N other users looking at this now!” widgets, an anchoring price with a slash through it next to a lower price, all that jazz. Agents should run the playbook for us.

[3] Agents also create giant volumes of unnecessarily defensive code, out of desire to defend against any conceivable ambiguity without knowing which risks are realistic.
