---
url: https://www.alistaircroll.com/updates/when-the-target-keeps-moving/
date_fetched: 2026-07-05
backfilled: true
---

# When the target keeps moving

When it's easy to code, scope creeps. But not all creep is bad. I analyzed a month of AI coding, and learned that we need to measure agentic software development differently.

Full disclosure: I've been working on an app for a few months, and will be launching it May 5. This has given me a lot of opportunity to understand how software development changes when working with AI agents. One of my agents (an OpenAI Codex agent) helped do the analysis this post covers, and wrote an early draft. I rewrote the whole thing; it was initially pretty academic, because the AI spent a bunch of time reading academic papers on software estimation and AI agents are what they consume. This agent is usually more fun, and says weird stuff like "more door than hinge" and "déjà-vu wearing a fake moustache" regularly.

The first progress report in April said we had 254 tasks. 140 were done, so we were 55% complete. A month later, we'd completed 1,257 tasks. Excluding cancelled work, we were 89.8% done. And yet there were still around 140 tasks left.

It had been a busy few months. What started as an idea has become a production-grade web application with audits, release gates, admin workflows, simulations, test harnesses, support tooling, deployment checks, and a long list of edge cases. Yet like Xeno walking by half-steps, the target kept moving with us.

That's because a project isn't a pile of known tasks that are being steadily shovelled from "not done" to "done." It's also a discovery engine. Every feature, test, audit, simulation, customer conversation, or UX change shows you the things you didn't know until they were staring you in the face.

Anticipating these things is the job of planning, specs, and project managers. Planning up front makes sense when the cost of actually building something is high. If you're building a bridge, you don't figure it out as you go along. You do things like requirement discovery, requirements, specification, and planning before you implement anything. You run simulations, build models, test your assumptions, and double-check your math. You update the plan and the schedule. Only then do you actually order the girders.

Software isn't a bridge. Getting it wrong is annoying, but generally isn't life-threatening. And if it is wrong, then you can fix it. You can make a copy, work on that copy, and when it's better, deploy the fix.

Despite this, the actual *coding* part of software has been relatively expensive, which means that we've always tried to get the plan right first.

Now that the cost of coding has plummeted, that's changing. AI agents don't just complete tickets—they inspect, decompose, test, question, file follow-ups, and expose new things you're going to need. *AI accelerates both delivery and discovery.*

In software engineering, there's a term for the never-ending growth of a project: Scope Creep. Agile software development has a caution against this, called YAGNI (You Aren't Gonna Need It)—you shouldn't build more than you need to. But the new economics of software mean scope creep isn't as an embarrassing exception to the plan—it's how you figure out what the plan was supposed to be all along. A growing backlog of features can be progress, but only if you distinguish creep that *reduces focus* from creep that *increases truth.*

## The data set

This project was a unique opportunity to analyze changes in scope first-hand because in addition to the features, we also built the tracking systems. Every task is recorded and explained and logged in exhaustive detail. Without giving too much away, the project is:

- A public web application with authenticated user flows.
- Backend APIs and data storage.
- An admin/operator surface.
- QA and production deployment paths.
- Automated tests, simulations, audits, and release gates.
- Agentic AI collaborators doing planning, coding, verification, and board hygiene.

This post analyzes exactly one month of data, using a bunch of local systems:

The data's not perfect. One of the AI agents deleted the task board by accident, and we had to rebuild it from specs and Github. AI Agents are forgetful toddler idiot savants and must be treated as such. I carry a daily backup on a keychain at all times. But we can still learn useful things.

## High-level project stats

Over the 32-day daily task window, the git log recorded 1,352 commits. The tiny social media platform we built so agents could coordinate with one another recorded more than 2,700 posts:

After spending a lot of time watching the next deadline stretch off into the horizon, I asked the agents to build me what I called an ahead/behind chart: How many tasks did we add, and how many did we complete? If we could stay mostly below that line, I figured, we'd be okay.

Here's what that looked like:

## Github tells us more

The task board tells us the names of the tasks; but Github worktrees tell us how those tasks moved through production. A lot of the project happened in short-lived topic branches and isolated worktrees. At the April 30 snapshot:

The 150 primary-remote topic branches were not evenly distributed by kind. Their names are too project-specific to publish, but the branch families are revealing:

The branches show us that the type of work shifted over time. Later work wasn't new product ambition—it was hardening, refactoring, verification, and system cleanup. Almost half of the merge commits in the month happened after April 21, the same period where the task board hovered around 90 percent complete while the active denominator grew by 367 tasks.

## The progress bar is a lie


Here are four recovered checkpoints from the month:

The project looks like rapid progress at first, followed by encroaching horror as you discover just how big the problem you set out to solve *really* is in the middle. Polish comes at the end.

Plenty of vibe coders are learning the hard way that like icebergs, 90% of the work is hidden.

Between April 21 and April 30 alone, we went from 1,032 to 1,399 tasks—367 newly visible active tasks. In that time, completed tasks grew from 921 to 1,257. That is 336 additional completed tasks. In other words, late in the project, **for every 100 tasks completed, roughly 109 new active tasks became visible.**

Rob Bowley used a coastline analogy in his 2011 essay on software estimation: the closer you measure, the more detail you see. Our early estimates described the coastline from the air, while late-stage QA walked the rocks. We didn't know what we didn't know, and delivery was also discovery.

## Discovery Velocity

Most project dashboards ask questions like, "how many tasks did we finish?" and "how much of the original backlog is complete?" But if you're building with AI , you need to track a slightly different metric:

Discovery velocity:how many new units ofwork became visible per unit of time.necessary

Delivery velocity without discovery velocity is misleading. A team can complete 40 tasks a day and still fall behind if it is discovering 45 tasks a day. And falling behind may be necessary, if the team is nevertheless moving from implementation to verification. It may even be a source of competitive advantage: you're discovering the moats and barriers to entry that make your software hard to copy.

More than half of our days were discovery days:

Those days coincided with major milestones like audits, big releases, security hardening, and so on:

The highest completion day, April 20, completed 132 tasks but also added 92. 40 tasks seems like a decent accomplishment, but from a project-learning perspective it was one of the most important days of the month: a lot of hidden work got named, and a lot of named work was retired.

## The last 10% was mostly not features

As of April 30, 5 groups account for 64% of all active unfinished tasks. Those groups are:

- Agent instruction refinement (getting agents who will use the platform to behave as expected.)
- Human testing and launch logistics (coordinating tests with human users and the launch team.)
- Simulation follow-up (analyzing the results of a test run and turning them into new tasks.)
- Load and scale validation (making sure the system can handle hundreds of users without breaking.)
- Onsite or operational readiness (preparing to run the thing in the real world.)

Metrics like "percent complete" treat all unfinished tasks as equal, but late-stage tasks are fundamentally different from early ones. While early-stage tasks ask, "can we make this thing?", late-stage tasks ask:

- Does it behave under stress?
- Can a real user recover when confused?
- Can operators see what is happening?
- Can we deploy safely?
- Can we explain the system to collaborators?
- Can we distinguish a product defect from a support issue?
- Can we trust the automation that says the build is healthy?

Late-stage tasks are also more entangled, and many of them require human involvement that means they can't be handled in overnight --dangerously-skip-permissions YOLO binges. They create new tasks faster because they operate on the whole system: a new feature touches a path, but a verification pass touches the whole product.

Fred Brooks argued in No Silver Bullet that the hard part of software is the specification, design, and testing of the conceptual construct, not just typing it into a programming language. AI agents change this. Cheap, uncomplaining coders turn design into discovery. If you ask a developer to rebuild the same thing ten different ways, they'll quit. Ask ten agents to do so and they'll just charge you more tokens. In some ways, this exposes gaps in thinking faster (but it also runs the risk of producing a product that works perfectly, but just for you.)

## Scope Creep versus Scope Discovery

Academic research into software development has often treated a growth or change in requirements as a measurable project dynamic. Ferreira, Collofello, Shunk, and Mackulak describe requirements volatility as growth or change during the development lifecycle and model its effects on cost, schedule, and quality.

Agentic development means that not all volatility is the same, and treating it as such is risky. Across over a thousand individual tasks, here are 7 different kinds, what they tell us, and how we responded to them:

Any task tracking board needs this classification, because a single "tasks added" number cannot tell you whether the project is drifting or learning.

## Agentic AI changes the shape of the curve

Classic software estimation literature already gives us plenty of reasons to distrust long-range precision. The planning fallacy literature distinguishes the "inside view" from the "outside view": if you only reason from the internal story of the prohect you're working on, you underestimate; if you compare against similar projects, you do better. Magne Jorgensen's survey of effort-estimation research argues that there is no single best estimation method, and that local historical data and tailored checklists improve estimates more reliably than generic models.

Agentic AI makes these findings more urgent. In our month of data, agents affected the curve in at least four ways.

- They **lowered the cost of decomposition**. A human might notice an issue and leave it as a vague concern. An agent can turn it into five tasks, wire dependencies, add tests, and update the board. The denominator grows because the work becomes countable.
- They **increased parallel inspection**. Multiple agents can inspect different surfaces at the same time: code, tests, UI, docs, security, deployment, and data flows. This raises discovery velocity.
- They **blended planning and implementation**. Traditional workflows often separate discovery, specification, and execution into rituals. In this project, an agent could discover a gap, write a short design, implement a fix, add tests, and file two follow-ups in a single session. That is productive, but it makes clean phase boundaries harder to observe.
- They **created measurement ambiguity**. METR's 2026 update on developer-productivity experiments notes that agentic tools make time measurement harder because developers may run agents concurrently, avoid submitting tasks they do not want to do without AI, and choose different kinds of tasks when AI is available.

The question is not just "how much faster did AI make implementation?" It is "how did AI change which work we attempted, which work we noticed, and which work became cheap enough to do?" An agentic project may produce *more finished* work and *more remaining work* simultaneously. The question is *did it produce a better product as a result?* For that, the jury's still out.

## Estimates affect the work

Estimates are not passive predictions. They change behavior. Jorgensen and Sjoberg's work on effort estimates found that overly optimistic estimates can lead to less effort and more errors in programming tasks, and that anchor values can affect estimates even when people are told the anchors are irrelevant. Jorgensen's later survey summarizes the broader point: estimates can be harmful when they become commitments too early or create incentives to fit work to the estimate.

Once the board says 90 percent, every new task feels like regression. It's what made me ask for the ahead/behind chart. But "90 percent complete" on a moving-denominator project does not mean "10 percent of the original plan remains." It means:

Of the work currently visible and not retired, 10 percent remains.

In our case, 90 percent complete meant the core product surface was mostly done, but confidence work was still creating tasks. The project was closer to launch, but also more honest about what launch required.

## A better dashboard

If I were designing the progress dashboard again, I wouldn't remove "percent complete." But I would demote it. Our latest iteration of the dashboard shows five different metrics:

- **Completed active tasks:**How much have we done so far?
- **Active remaining tasks:**How much is left to do?
- **Daily task additions by discovery type:**How much are we adding, and what sort of tasks are they?
- **Daily task delivery:**What did we get done today?
- **The discovery/delivery ratio:**Is the rate of things uncovered to things delivered improving?

You really need to track discovery (learning); delivery (completion) and how they're related (a sign of whether you're beyond the learning phase.)

For late-stage work, I would also track remaining tasks by type:

- Product functionality (this should show up at the start).
- Defect repair (how fast do you squash bugs, which is a measurement of your harness and development environment.)
- Verification (how well can you test what you've built, which is about automation and scripting.)
- Operations (once it's live, can it be operated?)
- Documentation and instruction (can other people take over/help out? Will support costs be high?)
- Human or organizational dependency (this increases as implementation gets closer.)
- Deferred / intentionally out of scope (this shows whether you're willing to cut.)

This lets you have better product conversations: "We have 142 tasks left" is emotionally heavy and operationally vague, but "we have 16 core-product tasks, 37 agent-instruction tasks, 31 validation tasks, and 18 human-test tasks" means something. I wouldn't plan the next agentic build as a fixed backlog with a burn-down chart, but instead, as three overlapping curves.

### 1. The first useful thing

Early on, optimize for coherent vertical slices. The first denominator is mostly fiction anyway. It is still worth writing down, but its job is orientation, not prediction. This is the new Minimum Viable Product. In this phase, you're tracking metrics such as:

- End-to-end paths.
- Core abstractions.
- Time to first working deployment.
- Early user (or agent) success.

### 2. Deliberately increase discovery

Once the system works, send agents and tests looking for trouble. This will make the board look worse, which is the point. You're trying to discover unknown unknowns and competitive moats. At this stage, that means data such as:

- Audit findings.
- Simulation failures.
- UX friction.
- Security and privacy issues.
- Operational gaps.
- Test unreliability.

This phase should have a high discovery velocity. Your backlog will grow; if it doesn't, you aren't looking hard enough.

### 3. Denominator stabilization

Only after serious verification should you expect the total number of tasks to stabilize. At that point, forecasts become more meaningful. Signs that this is happening include:

- Discovery ratio falls below 1 for sustained periods (you fix more than you find.)
- New tasks are increasingly deferrable rather than launch-blocking.
- Remaining tasks cluster around launch-related activities.
- Verification passes produce fewer novel classes of failure.
- The same issues stop reappearing under different names.

For this project, the late April data says we were not fully stabilized. From April 21 to April 30, the "remaining tasks" list grew by 367 while 336 tasks were completed. The project was making real progress, but still discovering at almost the same rate it was delivering. That's the number you should be discussing in a team meeting:

We are about 90 percent through currently visible active work, but our late-stage discovery ratio is still near 1. The launch question is whether the new discoveries are launch-blocking or deferrable.

## Is estimation broken in AI coding?

For years, software people have argued about whether estimation is broken, harmful, necessary, or merely misused. The moving denominator reframes the debate, because it's not just that humans are bad at estimating known tasks, it's that early in a project, *many of the real tasks do not exist yet as concepts*. They're latent in the system, and only appear when:

- A test fails.
- A user misunderstands a flow.
- An integration behaves differently in production.
- A security review asks a sharper question.
- A deployment path encounters an organizational constraint.
- An agent tries to follow instructions literally.
- A support workflow meets a real edge case.
- A performance run converts "probably fine" into numbers.

This is why agentic AI is such an interesting accelerant: *It helps us encounter reality faster.* If your management model expects the denominator to shrink smoothly, agentic development will look chaotic. If your model expects discovery and delivery to run together, the same data looks like learning.

- Good scope growth makes the project *more truthful.*
- Bad scope growth makes the project *less focused.*

You just need to distinguish which is which.

## References

- Frederick P. Brooks Jr., ["No Silver Bullet: Essence and Accidents of Software Engineering"](__https://cgi.csc.liv.ac.uk/~coopes/comp319/2016/papers/NoSilverBullet-Brooks1987.htm__), Computer, 1987.
- Rob Bowley, ["Estimation is at the root of most software project failures"](__https://blog.robbowley.net/2011/09/21/estimation-is-at-the-root-of-most-software-project-failures/__), 2011.
- Magne Jorgensen, ["What We Do and Don't Know about Software Development Effort Estimation"](__https://www.infoq.com/articles/software-development-effort-estimation/__), IEEE Software / InfoQ, 2014.
- Magne Jorgensen and Dag I.K. Sjoberg, ["Impact of effort estimates on software project work"](__https://www.sciencedirect.com/science/article/abs/pii/S0950584901002038__), Information and Software Technology, 2001.
- Susan Ferreira, James Collofello, Dan Shunk, and Gerald Mackulak, ["Understanding the effects of requirements volatility in software engineering by using analytical modeling and software process simulation"](__https://www.sciencedirect.com/science/article/abs/pii/S0164121209000557__), Journal of Systems and Software, 2009.
- Yael Grushka-Cockayne, Daniel Read, and Bert De Reyck, ["Planning for the planning fallacy"](__https://www.pmi.org/learning/library/2019/04/07/15/25/planning-fallacy-causes-solutions-project-expectations-6374__), PMI Research and Education Conference, 2012.
- METR, ["We are Changing our Developer Productivity Experiment Design"](__https://metr.org/blog/2026-02-24-uplift-update/__), 2026.
- Amy Deng / METR, ["Analyzing coding agent transcripts to upper bound productivity gains from AI agents"](__https://metr.org/notes/2026-02-17-exploratory-transcript-analysis-for-estimating-time-savings-from-coding-agents/__), 2026.
