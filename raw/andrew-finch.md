---
url: https://ai.engineer/code/2026/talks/andrew-finch
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

3:10:06Okay.

3:10:08All right. Can y'all hear me?

3:10:12>> All right. My name is Andrew and I think

3:10:16we're starting now. I don't know if

3:10:17there's a timer anywhere. Uh my name is

3:10:20Andrew and I'm excited to talk to you

3:10:22today about the safe software factory.

3:10:27Uh but first a little bit about me. Uh I

3:10:30am a staff software engineer at Sonar. I

3:10:34work on guitar which is Sonar's AI code

3:10:36review products. Uh and previously I was

3:10:40at uh Uber working on the driver trip

3:10:43experience and navigation.

3:10:45Uh and then after that I was at a health

3:10:47tech startup called forward for a little

3:10:49while uh building a sort of an AI

3:10:52enabled primary health care system.

3:10:59Okay. So the software factory this is

3:11:03like the hottest buzzword in our

3:11:06industry right now. Everyone's talking

3:11:09about software factories. Everyone's

3:11:10trying to build a software factory.

3:11:15And the question is, well, oh well, so

3:11:19first of all,

3:11:20um,

3:11:22most of you probably know what this is

3:11:25already. Um, but if you don't, I'll just

3:11:28give a really quick overview. Uh, this

3:11:31is sort of the the main software

3:11:33development life cycle loop uh, as it

3:11:36existed before the AI era. So we have uh

3:11:40inputs basically product specs, feature

3:11:42requests, tickets, things like that. Um

3:11:47we take those into the planning step if

3:11:49they're especially if they're

3:11:50complicated.

3:11:52Uh and then we then we build then we

3:11:54write the code

3:11:56uh and then we um uh put up a PR and

3:12:00that PR gets uh reviewed and it goes

3:12:04through CI and then it eventually merges

3:12:08uh and then we deploy that to production

3:12:11and then users interact with our

3:12:13software and they give us feedback or

3:12:15they find bugs

3:12:17um or we might also get some monitoring

3:12:20and alerts as well. And then those go

3:12:22back in as more inputs and the the cycle

3:12:26continues

3:12:28and and so um in the in the era of

3:12:32agentic coding and AI automation has

3:12:36followed every bottleneck. So the

3:12:38bottleneck used to be the the coding

3:12:40part the building um now with coding

3:12:43agents that's been automated. Um, and

3:12:47now the bottleneck has sort of shifted

3:12:49over to the that review step and we're

3:12:52starting to introduce uh AI code review

3:12:55systems for that. Um, and then and we're

3:13:00seeing uh AI systems and automation

3:13:02happen really at every step of this now.

3:13:05Uh so we're starting to see AI systems

3:13:08for monitoring deployments, controlling

3:13:11rollouts, triaging uh triaging, you

3:13:16know, tickets and incidents, planning.

3:13:18And so this this here is the sort of

3:13:21software factory that we're that we're

3:13:25creating.

3:13:28And and so the question that we all have

3:13:31now is how do we make this factory safe,

3:13:35right? Uh in other words, how can we

3:13:37introduce enough layers of verification

3:13:41into this so that we can trust that the

3:13:43software coming out of it is it works as

3:13:47expected. It's free of bugs, free of

3:13:49security vulnerabilities.

3:13:54Um, our CEO

3:13:56Tar said recently, "Code is abundant,

3:13:59confidence is not."

3:14:01So, what I'm going to talk about today

3:14:02is just how we at Sonar have been

3:14:04thinking about uh how to introduce uh

3:14:08confidence

3:14:10uh at at all sort of all different

3:14:12points within this software factory.

3:14:17And uh we're approaching this this

3:14:20challenge with a defense and depth

3:14:22approach that comes from adding more

3:14:26loops. You might not be surprised to

3:14:28hear

3:14:29and I'm going to talk about four of them

3:14:31today in particular. So there's the loop

3:14:34one which is the PR checks loop.

3:14:38Uh loop two uh which is the post deploy

3:14:42verifications loop.

3:14:45loop three uh which is the maintenance

3:14:48loop

3:14:50and loop four which is the knowledge

3:14:52loop and this is kind of a kind of a

3:14:54special loop. Uh so first I'm going to

3:14:57talk about loop one the PR checks loop.

3:15:01So this is the loop that we probably all

3:15:02have in some form. Uh you put up a PR,

3:15:06it goes through some CI checks, it gets

3:15:08a code review. Nowadays it's probably

3:15:10getting an AI code review as well. Um,

3:15:12but we're starting to see a lot more

3:15:14automation being introduced here as

3:15:16well. So, we're starting to see agents

3:15:19pushing fixes to respond to the code

3:15:21review comments. Um, and this loop kind

3:15:25of goes around and round until the PR

3:15:28turns green.

3:15:34And there's actually a lot more that we

3:15:37can be doing here. um because catching a

3:15:40problem at this point is still cheaper

3:15:42than after it merges into the codebase.

3:15:45Uh so we have a real opportunity here to

3:15:47widen this review past the sort of

3:15:50normal tests and code review that most

3:15:53of us do. So of course we have AI code

3:15:57review. We can also do static analysis

3:16:00here. Many of us do that.

3:16:03Uh we could do what's called functional

3:16:05validation. So this is where we can look

3:16:07at uh what the ticket asks for and see

3:16:09if the PR actually addresses those

3:16:12things.

3:16:15Um we can look at rules that the team

3:16:19may set for coding conventions

3:16:23uh see if best practices things like

3:16:26that see if the PR is adhering to those.

3:16:30We can look at knowledge. So this can be

3:16:32knowledge accumulated from company

3:16:34documentation,

3:16:36from things that have been learned on

3:16:38past code reviews or past outages.

3:16:43Uh we could do cross repo checks. So we

3:16:45could actually look at other

3:16:46repositories and see if anything in in

3:16:48this PR uh will break or cause something

3:16:52to break in another repository.

3:16:56Uh we could do cross PR analysis. So, we

3:16:59can look at all the other open PRs as

3:17:01well and see if there's anything uh in

3:17:05this PR that conflicts with one of

3:17:07those.

3:17:10We could do security review, check for

3:17:12vulnerabilities, secrets, dependencies.

3:17:17We could do endto-end testing. So, for

3:17:19example, if this is a web app, we could

3:17:20open up in a browser and and click

3:17:22around and see if it works as expected.

3:17:27Um and finally we can do risk

3:17:30assessment. So we could sort of uh look

3:17:32at this and compute uh the blast radius

3:17:36and sort of categorize this into like

3:17:38low, medium or high risk. And that can

3:17:41actually help us determine if this is a

3:17:43PR that a human even needs to look at at

3:17:45all.

3:17:47And so all of these things can flow into

3:17:49this the sort of decision this merge

3:17:51gate this decision of whether this PR is

3:17:53mergeable. And of course uh every time a

3:17:57fix is pushed all of these run again at

3:18:00least in some degree.

3:18:04And and with all of this it's important

3:18:06to gate the that merge decision not just

3:18:09on the fact that a test ran or that it

3:18:11passed but on the evidence. So that

3:18:13would be things like the actual code

3:18:15review findings, the results of the

3:18:17quality gates,

3:18:19uh any vulnerabilities,

3:18:22um the risk level. So all of that can be

3:18:26uh basically fed into this merge gate

3:18:29and checked against criteria that the

3:18:31team can set for what is a mergeable PR.

3:18:35And when you record that evidence uh to

3:18:38the PR, it just makes this uh the merge

3:18:41gate much stronger.

3:18:44And so what we're seeing now uh with

3:18:47teams who do this well uh is that they

3:18:49can actually delegate more and more

3:18:51trusts uh to this type of automation on

3:18:54the PR review. Uh so we're seeing teams

3:18:58begin to turn on not just automatic

3:19:00merge blocking but uh even auto approve

3:19:05in some cases for low-risk changes and

3:19:07and even automerge for certain types of

3:19:09PRs that are that are lowrisk and

3:19:11approved

3:19:14and this can just free up a lot of

3:19:15review bandwidth for human reviewers to

3:19:17review the the more high-risisk changes.

3:19:22Okay. So now um I'm going to talk about

3:19:25loop two which is our post deploy

3:19:27verification loop. Uh so this is I would

3:19:32say very much in its early stages right

3:19:34now. People are experimenting with all

3:19:36kinds of things here. Um but the idea is

3:19:39that once your code is deployed uh you

3:19:42can have you you can build AI systems to

3:19:45check uh to read like production logs,

3:19:48traces, metrics and see if your code is

3:19:50actually behaving as you expect it to in

3:19:53production.

3:19:55And then from there you can actually um

3:19:59uh we're starting to see things like

3:20:00agentic rollout where agents can decide

3:20:03whether to widen a roll out or roll back

3:20:06turn a feature flag off

3:20:09and you can also do uh AI outage

3:20:11monitoring. So you can uh monitor alerts

3:20:15and uh use AI systems to help you uh

3:20:19root cause and fix outages.

3:20:24Okay. Uh so moving on to loop three. Uh

3:20:29so this is what I call the maintenance

3:20:30loop. Uh this is kind of a a parallel

3:20:33loop that runs alongside your main

3:20:36software development uh life cycle loop.

3:20:39And this is where we continuously scan

3:20:41the codebase on some schedule. And we

3:20:44check for a lot of the same things that

3:20:45we check for on the PR. We we could do

3:20:47code review, static analysis, security

3:20:49checks, etc. And then we can if we find

3:20:53an issue, we can open up a remediation

3:20:55PR to address that. And this lets us

3:20:59catch things that we couldn't catch in

3:21:01the PR. So for example, you could

3:21:03imagine

3:21:04um maybe a new rule was introduced after

3:21:07code merged or a new vulnerability was

3:21:10published after code merged.

3:21:13um or even something like slow drift

3:21:16where you you slowly get duplication or

3:21:19complexity building up that you can't

3:21:21really see on a single PR. Uh you can

3:21:24catch these kinds of things here

3:21:27with these uh with the continuous

3:21:28codebase scans.

3:21:33Okay. Okay. And so finally we move over

3:21:35to the knowledge loop and and this is

3:21:38becoming we're seeing this become a very

3:21:39important uh loop for a lot of

3:21:42companies. Um every company has a lot of

3:21:46knowledge documentation product specs

3:21:50um business decisions

3:21:54um and the agents do a better job if

3:21:58they have that knowledge. Right? So

3:22:01[clears throat] this loop basically

3:22:03collects all that knowledge from

3:22:04different sources as well as things that

3:22:06were learned on prior code reviews and

3:22:09from prior outages. It's curated and

3:22:13then it can be uh fed into all of the

3:22:16other loops, all of these other agents

3:22:19um so they can uh do a better job.

3:22:25And and for example, this is really

3:22:27powerful with your coding agents because

3:22:30if they have some of this knowledge,

3:22:32they can probably do a better job the

3:22:33first time, fewer issues.

3:22:37Um, and then you'll you'll wind up with

3:22:39basically fewer review laps at the uh

3:22:42review stage.

3:22:47Okay. So, what does this mean for our

3:22:51role as human software engineers in this

3:22:54factory? Uh, you might be asking, okay,

3:22:57so if if we nail all of this, can we

3:22:59just uh turn the lights out and run a

3:23:02dark factory?

3:23:04Uh, and I would say probably not. Not

3:23:06right now. Uh, I like to say the factory

3:23:09runs dim, not dark. It's still very

3:23:12important for us to [cough]

3:23:15intervene in places where our expertise

3:23:18is is really important.

3:23:20Uh so that would be things like auditing

3:23:23risky code.

3:23:25Um verifying that evidence, verifying

3:23:29things are actually working as expected.

3:23:32Handling exceptions like rollbacks and

3:23:35incidents.

3:23:37Uh reviewing architecture.

3:23:41Agents are not very good at this yet, I

3:23:43would argue.

3:23:46um and supplying the why or or helping

3:23:49to build that knowledge that that powers

3:23:51all these agents and all these loops,

3:23:54supplying the product intent, the

3:23:56trade-offs, the business decisions,

3:23:58things like that.

3:24:01And of course, we engineer the factory

3:24:04ourselves, right? We're we've sort of

3:24:06moved into this era where we're we're

3:24:08building the software factory as a

3:24:10whole.

3:24:13So with that, I hope that gave you sort

3:24:17of a framework for how to think about um

3:24:20not just how to build your software

3:24:21factory uh but how to build a safe

3:24:24software factory. So thank you. That's

3:24:28all for me. [applause]
