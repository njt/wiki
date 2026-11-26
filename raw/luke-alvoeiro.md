---
url: https://ai.engineer/code/2026/talks/luke-alvoeiro
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

3:42:45All set.

3:42:47>> Hi everyone. Uh I'm Luke. I lead our

3:42:50core agent team here at Factory. Uh

3:42:52today I want to talk about autonomy and

3:42:54why uh the teams that are getting the

3:42:56most out of coding agents are not

3:42:57actually the ones that are handing over

3:42:59like the most work on day one. It's

3:43:00instead the folks who are ramping up

3:43:02autonomy gradually, one checkpoint at a

3:43:05time until what they have starts to

3:43:06resemble a software factory.

3:43:09So for those who don't know us, a

3:43:11factory is a platform uh that builds

3:43:13droid and then a a whole like surface

3:43:15for software engineering and building

3:43:17software on top of that coding agent.

3:43:20Hundreds of thousands of developers use

3:43:21us around the world at companies like

3:43:23Nvidia, T-Mobile, uh and Adobe. What

3:43:26these companies engineers are building

3:43:28with us is is really like a software

3:43:30factory, a system that runs 24/7 and

3:43:33builds software under the company's

3:43:34control. And that's a loop that takes in

3:43:38signals from wherever information is

3:43:39already available, turns them into

3:43:41changes, validates them, uh, and then

3:43:43learns from what production shows. A

3:43:45signal is really like anything that

3:43:47tells you how your software should

3:43:48evolve. So that could be, you know, a

3:43:50sentry alert or someone's idea on Slack.

3:43:53Every signal goes through the same

3:43:54stages. So you standardize you

3:43:56standardize the the process not the

3:43:58output.

3:44:00So we built that platform on five

3:44:02foundations. It has to be transparent,

3:44:05modular, governable, efficient and

3:44:08frontier.

3:44:10Most people picture a software factory

3:44:12as this kind of like lights out building

3:44:14where uh there's no people inside of it.

3:44:16Requirements come in one end and merged

3:44:18PRs come out the other. And and really

3:44:21what we saw firsthand in the field is

3:44:23that that's not the case. The best ones

3:44:24aren't set up like that. They're built

3:44:26around people instead. Uh, humans decide

3:44:29what matters and how much to trust

3:44:31droids over time. And those droids earn

3:44:33their keep as they deliver more work

3:44:34that's higher quality.

3:44:36Every company's software factory looks

3:44:38very different to each other. So, what

3:44:40we what we sell is not really this

3:44:41factory, it's the tools and the building

3:44:43blocks to compose them.

3:44:46Now, it's it's worth kind of looking at

3:44:48how we got here in the first place. Back

3:44:50in April, one of our flagship products

3:44:51was Missions, which was one of like the

3:44:54first multi- aent uh products to really

3:44:56take like a bite out of the human

3:44:58attention bottleneck that existed in in

3:45:00software engineering. You'd describe a

3:45:02goal, you'd like argue with the

3:45:04orchestrator until you were aligned,

3:45:06you'd approve the plan, and then it

3:45:08would go handle implementation for like

3:45:10days or even weeks. When the goal was

3:45:12clear and verifiable, you really could

3:45:14come back to like completed work pretty

3:45:16consistently.

3:45:18But as more teams adopted it, we heard

3:45:20some consistent like pieces of feedback.

3:45:22The first was how missions asked you to

3:45:24work. The product assumed that you could

3:45:26plan everything up front and then it

3:45:27would just work. Uh and the UX sort of

3:45:29suggested that you shouldn't interrupt

3:45:31them. Missions didn't really like ask

3:45:32you any questions or provide like an

3:45:34interactive surface for that uh

3:45:36communication between you and the agent

3:45:38once it was starting implementation. So

3:45:40this suits like migrations and big uh

3:45:43big coding like feature projects but it

3:45:45didn't really fit fit the bill for like

3:45:47most kinds of other work.

3:45:49The second was cost and time and that

3:45:52came from honestly like the models

3:45:54getting really really good. I'm sure

3:45:56you've all seen now adversarial review

3:45:58is really really good at finding these

3:46:00issues in your code. And so, uh, at the

3:46:02time we were adding adversarial review

3:46:05into every milestone's boundary. That

3:46:07just kind of like grew the scope of that

3:46:09mission from something like 60 features

3:46:12to over 200. And so you ended up having

3:46:15a token bill and a timeline that matched

3:46:17that. And then the last one was that

3:46:20missions were keeping their context to

3:46:21one person. So that was fine for like a

3:46:24single project, but it meant that the

3:46:25rest of the team couldn't really build

3:46:27on what that mission had learned.

3:46:30So we went back to the drawing board and

3:46:32one mental model kept on coming up. When

3:46:34you have like an org that is running

3:46:36dozens or hundreds of agents at a given

3:46:39point in time, you actually have like a

3:46:41lot of the the problems that operating

3:46:44systems back in the day solved. How to

3:46:46schedule work, how to run what many

3:46:48things at once safely, and how to share

3:46:50state.

3:46:53So every operating system starts with

3:46:55the thing that actually does the work

3:46:57and we wanted to revisit the agent

3:46:58itself. Long horizon agents run for

3:47:01thousands of turns and every

3:47:03inefficiency across of those gets

3:47:05multiplied. So you could have, you know,

3:47:07an extra inference request that's

3:47:08unnecessary or a a tool result that's

3:47:11pretty bloated or a prompt cache miss.

3:47:13All those things compound over time and

3:47:16lead to waste.

3:47:18So we decided to make some pretty

3:47:19significant changes to the harness

3:47:20itself. The biggest one was programmatic

3:47:23tool calling, also known as code mode.

3:47:25Instead of calling one tool, waiting on

3:47:28the model, and then calling the next, we

3:47:29have the model write a small program

3:47:31that executes all those tools in one go,

3:47:33and then prints out only what's

3:47:35relevant.

3:47:36So there's this example here where you

3:47:38have 150 linear tickets that are getting

3:47:40moved from one project to another. done

3:47:43the normal way that's like 30 inference

3:47:44requests, a million tokens, but as one

3:47:47model generated program, you actually

3:47:49only have like 60k tokens. It runs much

3:47:51faster and the model only sees what is

3:47:53actually relevant to the task that the

3:47:55the fact whether it was like successful

3:47:57or not.

3:47:59So this layer of the OS is where being

3:48:00frontier and efficient matters the most.

3:48:03A new model lands every few weeks and no

3:48:05two fail fail the same way. So we tune

3:48:07our harness in such a way that evens out

3:48:09this jaggedness across all these

3:48:11different models. The result is that

3:48:13Droid is now, you know, higher quality

3:48:15at a lower cost than every other coding

3:48:17agent on the market.

3:48:20Now you have a faster agent, but that

3:48:21that doesn't really give you like a step

3:48:23function increase in in how much work

3:48:25gets done inside of your system. For

3:48:27that, you need many agents running at

3:48:29once, and they need somewhere to run.

3:48:31remote missions were kind of held back

3:48:33that by this like fundamental limitation

3:48:35in the system where there was one

3:48:37computer that handled all the work. But

3:48:39to run a software factory, you need uh

3:48:41compute that scales with the volume of

3:48:43work in the system. Otherwise, every

3:48:45agent is is fighting for the same files

3:48:47or system resources. And so that's why

3:48:50we build environments which are reusable

3:48:52definitions of the sandboxes that your

3:48:53droids need, complete with the repos,

3:48:56tools, and dependencies that they that

3:48:58they need to get started with. The

3:49:00system is governable. So there's one

3:49:02comp control plane that orchestrates all

3:49:04of these sandboxes together wherever you

3:49:06need them. They can run inside of your

3:49:08own VPC on on EC2, Kubernetes, whatever

3:49:12compute you already operate. So your

3:49:14code, your data and your network are in

3:49:16your control. Compute becomes pluggable

3:49:19just like we made the BY options and

3:49:21automodel uh options that we provide uh

3:49:24make that provide sort of like this like

3:49:25model pluggable layer. So now we had

3:49:28agents that were cheap enough to run all

3:49:30the time and the compute to run many at

3:49:33once. And so then the next question was

3:49:35how do we orchestrate all of this work?

3:49:38At first we tried making like this DAG

3:49:40very complex system of agents composed

3:49:43together by like these very like strong

3:49:45rules and it didn't really work. You end

3:49:47up having a lot of the same problems

3:49:49that plagued missions uh in the first

3:49:51place because it was too rigid and as a

3:49:53result it was too expensive to run and

3:49:55it didn't generalize well.

3:49:57So what we landed on instead is

3:49:59something that we call a workstream.

3:50:01It's a standing goal with a team of

3:50:03agents to support it. It has this goal.

3:50:05It has an identity, a an environment

3:50:07that it runs on and memory.

3:50:10A new workstream runs as you and it has

3:50:13your access. When it outgrows one

3:50:15person, it can be moved to a service

3:50:17account with its own credentials. So you

3:50:19stay on as a member, but the work keeps

3:50:21h happening under an identity that the

3:50:23the org actually controls.

3:50:26At the center of every workstream is an

3:50:28orchestrator and its job looks a lot

3:50:30like a scheduler in an OS. Unlike the

3:50:33graph that we started with, it assembles

3:50:35its agents in whatever way is most

3:50:37natural for the model. So the

3:50:38instructions are pretty short, right?

3:50:40It's like uh delegate first. Give each

3:50:43agent an area of ownership and then for

3:50:45tasks that can fall into an existing

3:50:47area of ownership, delegate to those.

3:50:50You start by talking to it. uh you just

3:50:52kind of hand things over as you think of

3:50:54them and then it fans out the work so

3:50:56that then uh you can get the the control

3:50:58back to you. Uh agents report back when

3:51:01they finish, when they get blocked, when

3:51:03there's something that changes the plan

3:51:04overall.

3:51:07But the work stream doesn't just stop

3:51:08there because it takes on more on its

3:51:11own and it learns from your habits. At

3:51:13some point the orchestrator may ask you

3:51:14how you decide what to work on and it

3:51:16learns that maybe bugs are coming from

3:51:18two Slack channels.

3:51:21So here it can set up an intake and a

3:51:23triage listener. Every 30 minutes it can

3:51:25go in and pull in those reports from

3:51:27those channels, ddup them against you

3:51:30know existing linear issues or uh sentry

3:51:33sentry alerts that already exist and

3:51:34then weigh each of those against the

3:51:36workstream's goal. So a Slack complaint

3:51:39and a customer error report become one

3:51:41change. The output of triage is a spec

3:51:45that you review and uh you're capable of

3:51:48like weighing in on before any code gets

3:51:50written. Once you approve a spec, the

3:51:53same worker opens a PR, stewards it

3:51:56until it's merge ready uh and then make

3:51:58sure that you know review uh comments

3:52:00and CI are passing.

3:52:03So each one of those stages actually

3:52:05maps to an autonomy level in this

3:52:07system. At L1, the workstream posits for

3:52:10human review at the spec creation, at

3:52:13the PR, and before hitting merge. But

3:52:17you know, as you level up that th those

3:52:19levels of autonomy, you can get to L4

3:52:21where you're automerging everything by

3:52:23default. The owner gets to set the

3:52:25level, but the confidence to raise it

3:52:27comes from watching the work. So when

3:52:30proposed changes are accurate or PRs

3:52:33consistently merge at the quality that

3:52:34you'd expect, that's actually your cue

3:52:36to move to the next checkpoint. And so

3:52:38that becomes the dial for you. You don't

3:52:40need to decide on day one whether you

3:52:42trust a coding agent to own, you know,

3:52:43your like CI health workstream off the

3:52:46bat. You just need to trust it to to

3:52:48like unlock the next checkpoint.

3:52:52So in a factory, an agent that starts

3:52:54from scratch repeats the same mistakes

3:52:56the last one already did. And so we

3:52:58built workstreams to have a shared file

3:53:01system. Agents are are encouraged to

3:53:04keep memory and version skills and so

3:53:07their PRs get better over time. They're

3:53:09also proactive and they go looking for

3:53:11context that they're missing. So for

3:53:13example, say that your work notices that

3:53:15other people are shipping related

3:53:17changes. It sets up an automation that

3:53:19pulls in their PRs,

3:53:22analyzes what they've been working on

3:53:23and what comments they've been receiving

3:53:25in those PRs, and then incorporates them

3:53:27into their memory and skills. So, what

3:53:30one agent learns, others can use, and

3:53:32nothing starts cold.

3:53:36That folder is also how agents work

3:53:38together on a project. When the

3:53:40orchestrator spawns an agent, it's

3:53:42capable of telling, you know, each agent

3:53:44where does it where where does its

3:53:46result sort of belong inside of this

3:53:48shared folder. And so every agent can

3:53:51read and write to the same place. If you

3:53:54ask it a workstream why checkout got

3:53:56slower, for example, it can send dozens

3:53:59of agents out to track down all those

3:54:01different leads that you have. And then

3:54:02ultimately each one can write its

3:54:05findings to the folder and the

3:54:06orchestrator can merge them.

3:54:09Now you still need a barrier between

3:54:10implementation and merging. And for us

3:54:13that barrier is platform level

3:54:14validation. Every PR gets code review,

3:54:17security review, and dedicated QA

3:54:20against the live preview. And so once

3:54:22every review thread is resolved, a risk

3:54:24judge then goes in and decides whether

3:54:26it's safe to approve or not. The judge

3:54:28gets the exact if and a check out of the

3:54:30branch so it can like build and test it.

3:54:32Uh but ultimately it reads its policy

3:54:34from our mainline branch so that the

3:54:36change can't rewrite the rules that it's

3:54:38being judged by.

3:54:40Every week a droid minds the review

3:54:43comments and goes through figures out

3:54:45which one authors have actually acted on

3:54:47and then pulls them pulls those into

3:54:49like review guidelines so that the the

3:54:51loop sort of improves over time. We also

3:54:54have droids that every day run and

3:54:56analyze reverts, hot fixes, new sentry

3:54:59errors, for example, and traces them

3:55:01back to the PRs that that caused them.

3:55:03And so that also is a signal into this

3:55:05same loop.

3:55:07What we're really hell climbing on here

3:55:09is the regretfree merge rate, the share

3:55:12of changes that merge and never need a

3:55:14revert or a follow-up fix. We measure it

3:55:17for droid ridden and humanridden PRs

3:55:19side by side. And it's the same way that

3:55:21we let like self-driving cars onto the

3:55:22road. We didn't just give them the green

3:55:24light. We only uh did so once they

3:55:27crashed less often than human-driven

3:55:29ones did. And so once AI written changes

3:55:32break things less often than human ones

3:55:34and you can actually measure that then

3:55:36you can let them merge and that's the

3:55:38bar for turning a workstream to L4.

3:55:41Workstream is now like the platform that

3:55:43we build new features on top of. So

3:55:45migrate is the first one. It sort of

3:55:47fulfills a niche in our product um like

3:55:50framework for for what missions used to

3:55:53be this like dedicated oneshot migration

3:55:55experience and continuous remediation is

3:55:57the second where it follows that

3:55:59circular shape that workstreams are best

3:56:01suited for because there the work never

3:56:02ends.

3:56:06We run factory uh on top of workstreams

3:56:08ourselves and so this is what it

3:56:10actually looks like in production. In a

3:56:12weekend this past October, we shipped

3:56:14about 800 poll requests in our dev

3:56:16branch and Droid wrote 99% of them and

3:56:19reviewed all of them. Now, work landed

3:56:21in one of two ways. It was either a

3:56:23human picking up a signal manually uh or

3:56:26increasingly nowadays, it's a workstream

3:56:28picking it up uh automatically triaging

3:56:31it against the goal of the workstream

3:56:33and then ultimately writing a spec and

3:56:35merging it.

3:56:3752% of PRs that we ship right now are

3:56:40coming from workshops themselves and

3:56:41some of these are operating on full

3:56:43autonomy. So for example, the one that

3:56:46uh fixes bug reports from a dedicated

3:56:48Slack channel merges 53 PRs per week.

3:56:50Another one is our flaky test work

3:56:52stream which merges about 40.

3:56:56Fully autonomous PRs aren't really the

3:56:59only thing worth tracking though. It

3:57:00what matters a lot more is the

3:57:02progression. how much of your work flows

3:57:04through work streams and how many

3:57:05checkpoints does each one still need. As

3:57:08more work flows unrestricted, your

3:57:10engineers become the regulators of this

3:57:12system. They decide, you know, what

3:57:14signals to watch, uh what are the goals

3:57:16and the owners of those, what are the

3:57:18skills and review guidelines that you

3:57:19need in order to keep this code high

3:57:21quality and how much autonomy each

3:57:22workstream gets.

3:57:24And so there's just one question that

3:57:26tells you, you know, whether whether you

3:57:28have a software factory or faster

3:57:30engineers, and that is, can you name and

3:57:32track your signals? Every team already

3:57:34has them. You have bug reports in Slack,

3:57:37you know, failing CI jobs, error spikes,

3:57:40tickets that nobody ever picks up, but

3:57:42most of them wait on a person to notice.

3:57:44And so my ask to all of you is to write

3:57:47them down and then pick the one that you

3:57:48waste the most time on. give it to

3:57:51workstream on L1 and then set up our

3:57:53validation experience and then you're

3:57:54well on your way to building software

3:57:56that improves itself with all the

3:57:58signals that you choose to track. Thanks

3:58:01everyone and I'll be around to answer

3:58:02any questions today. [applause]
