---
url: https://ai.engineer/code/2026/talks/lauren-tan
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

1:01:02Hello everyone.

1:01:04Don't know if you can see my slides, but

1:01:06uh my name is Potato, also known as

1:01:09Lauren, and I'm very excited to chat

1:01:11with all of you today. Maybe you can get

1:01:13a big round of applause for our tech

1:01:15team helping me with my laptop.

1:01:17[applause]

1:01:24Good.

1:01:27I don't see it.

1:01:34>> Ah, there you go.

1:01:37All right. Um, so again, I'm my name is

1:01:40Potato. Uh, I work on Grabbot at

1:01:43SpaceXsi.

1:01:45And can I just get a quick show of

1:01:46hands? How many of you have ever looked

1:01:49at code that somebody else wrote and you

1:01:52thought that's all slap?

1:01:54>> [laughter]

1:01:55>> Well, I have too,

1:01:59but today I'm going to convince you that

1:02:00you can turn that slop into gold.

1:02:06Uh, and so I did something crazy last

1:02:08month and I shipped 3,000 ploy crust to

1:02:11production. Now, before agents, we never

1:02:14really cared about volume. Uh, except

1:02:16for the outliers.

1:02:18And at my previous company, we called

1:02:20these people coding machines. people

1:02:22like Brett Taylor who famously rewrote

1:02:24all of Google Maps uh in the weekend

1:02:27just an insanely productive engineer. Uh

1:02:30and well with agents anyone can now be a

1:02:32coding machine. And my thesis is that

1:02:35the job of a software engineer the meta

1:02:37of being a software engineer is figuring

1:02:39out how to maintain or even improve the

1:02:42quality of your code while massively

1:02:44increasing your productivity.

1:02:47To get there, some people like to call

1:02:49this a software factory. Uh, but I like

1:02:51to keep it simple and I think of it as

1:02:54all about trust.

1:02:56Not blind trust, but investing in your

1:02:59agents environment so you can trust that

1:03:01your agents are going to do the right

1:03:03thing. Now, it takes quite a lot of

1:03:06effort to get there. Um, and it took me

1:03:09six months to get to a point where I

1:03:11could ship thousands of pull requests, a

1:03:13lot of them in my sleep, but it's

1:03:15certainly possible and it will take you

1:03:18much less time than I did. So let's

1:03:20start by talking about how we move from

1:03:22a low trust environment to a high trust

1:03:25one.

1:03:27So this inverse pyramid is how I think

1:03:29about where to spend time on your agents

1:03:32environment.

1:03:34The first two layers are deterministic.

1:03:36So using your code itself. So things

1:03:38like good API design, architecture,

1:03:41refactoring or using lint rules, type

1:03:44systems, compilers or you know different

1:03:46kinds of testing. And you can use both

1:03:49of these layers together to constrain

1:03:51what agents can do and force them to do

1:03:54the right thing by default.

1:03:56Verification, both runtime and formal,

1:03:59are great tools to layer on to find bugs

1:04:02and check properties and the invariance

1:04:04in your system that you want to hold

1:04:05true and also help you check whether or

1:04:08not the agent understood your goals and

1:04:10your intent.

1:04:12And then finally, you want to layer with

1:04:14a little bit of pros. So things like

1:04:16rules, skills, agents.mmd, things that

1:04:20require judgment.

1:04:23But let's start with your codebase.

1:04:25Agents generate their outputs based on

1:04:27their inputs. And so the the quality of

1:04:30the input really matters. And it just so

1:04:32happens that your codebase is one of the

1:04:34largest inputs. So whenever you're

1:04:37correcting your agent, think about how

1:04:39you can make that a deterministic check

1:04:41instead of adding another line to your

1:04:43rules or your agents.mmd because it's

1:04:46far more reproducible when it's in code

1:04:48and it lets you trust it more.

1:04:51And the analogy that I like to use is

1:04:53that your codebase is kind of like a

1:04:55garden. You know, code used to arrive a

1:04:58few pull requests at a time. And uh I

1:05:01remember going into work in 2024 and

1:05:04reviewing the three pull requests that

1:05:06my colleagues wrote colleagues wrote and

1:05:08today that number is more like 300. So

1:05:12uh you know every engineer is now a team

1:05:14of hundreds or even thousands of agents

1:05:17and code is arriving so rapidly that it

1:05:20almost feels like a continuous stream

1:05:22and it almost feels organic like a

1:05:25living thing.

1:05:27And in this analogy, your codebase, the

1:05:30constraints that you impose, the

1:05:32guardrails that you put up are the

1:05:34concrete foundations that the new code

1:05:36that your agents write grow around just

1:05:39like uh the building in this picture.

1:05:42And so in many ways, the job of being a

1:05:43software engineer is kind of becoming

1:05:45like gardening

1:05:48because code is the biggest input for

1:05:50agents. It means that any workarounds um

1:05:54anti-atterns deprecated code tech is now

1:05:58likely to be just copied by the next

1:05:59agent since it looks like the

1:06:01established pattern in your codebase and

1:06:04uh it's like you have some weeds or

1:06:07pests growing in your garden and then

1:06:09they spread everywhere before you know

1:06:11it.

1:06:15So you want to make your codebase as

1:06:17pristine as possible so that agents are

1:06:19always copying the good stuff. All

1:06:22right, let's look at some real examples.

1:06:24Uh so Dune is a client framework that we

1:06:27created for graphbot to implement this

1:06:29idea for organic architecture.

1:06:32It's not really that interesting

1:06:33technically. Uh you can kind of think of

1:06:35Dune as um kind of like Nex.js for

1:06:38Electron apps. But the interesting part

1:06:40of Dune is that it has a lot of

1:06:42constraints and lit rules that we've

1:06:44encoded uh a lot of the lessons that

1:06:46we've learned building desktop

1:06:48applications.

1:06:50And one such lesson is that anything

1:06:52that requires nuance to use well often

1:06:55leads to slop. Uh you know your classic

1:06:58example is uh you know use effect in

1:07:00React, right? It has its use cases uh

1:07:03but it's often misused. You know some

1:07:05people joke that you need a PhD to use

1:07:07use effect. Um and in in other words, it

1:07:10really requires a lot of nuance to use

1:07:13ball and agents are not good at using

1:07:15it. Um so our solution has been to just

1:07:19ban it in product code and it's really

1:07:21only allowed in some places inside of

1:07:24Dune APIs

1:07:26and effectively this problem has now

1:07:28been categorically eliminated and we

1:07:31never need to catch that during code

1:07:32review again.

1:07:34Another interesting pattern that we use

1:07:36is context through directory structure.

1:07:39Uh so one antiattern that we saw with

1:07:41cursor which is an electron app is that

1:07:44agents would often import main thread

1:07:46code into the renderer thread which

1:07:48would cause long tasks and jank.

1:07:52So in uh grapot and in dunes we've made

1:07:55this very very conventional. Uh we have

1:07:57these two top level directories

1:07:59literally called electron renderer and

1:08:01electron main and imports between them

1:08:04are forbidden or banned by our

1:08:06dependency graph analyzer and some lint

1:08:09rules. And again you know these are

1:08:11simple lint rules to set up but it means

1:08:13one less thing that we need to review

1:08:15and think about uh you know during code

1:08:18review.

1:08:20And then one more surprising thing that

1:08:23I've seen is agents using markdown. So

1:08:26things like documentation, uh, code

1:08:28comments, rules or skills, and they use

1:08:31this as a place to write down their

1:08:33justifications for not doing the right

1:08:35thing. Or, I don't know if you've ever

1:08:37seen this, but they'll over rotate on

1:08:39some small correction that you made, and

1:08:41they treat it like a hard rule that can

1:08:43never be broken ever again.

1:08:46And here's another example from cursor.

1:08:49So agents really loved using imperative

1:08:51timers. So things like set timeout,

1:08:53request animation frame, request idle

1:08:55call back. And they were using these as

1:08:57quick fixes because they were really

1:09:00patching over an architecture clash

1:09:02between uh VS Code's imperative style

1:09:05and React's declarative style.

1:09:08So our choice of architecture actually

1:09:10made both the client and the agent

1:09:13performance much worse. And if you've

1:09:16ever done any performance work in

1:09:17JavaScript, you'll know that when you

1:09:18have tons of timers being rapidly

1:09:20created and destroyed causes a lot of uh

1:09:23major garbage collection, which causes a

1:09:25lot of lag and jen. Uh but the worst

1:09:28part is that agents wrote a lot of code

1:09:30comments like you can see in red here

1:09:32explaining why this quick fix was

1:09:34justified and before you know it, we had

1:09:38timers spreading everywhere and the you

1:09:40know the performance of the app really

1:09:42suffered.

1:09:44Now in Grockbot uh we learned our lesson

1:09:47there and we decided to just ban 99% of

1:09:49comments uh because they go stale

1:09:52easily. Uh you know humans just really

1:09:54don't bother reading the comment slop

1:09:56anyway. And again it gives an agent the

1:09:59an escape hatch to justify not doing the

1:10:02right thing because it's hard. Um and

1:10:05the worst part about comments though is

1:10:07that they're not enforcable. So what

1:10:09seems well-intentioned, you know, like

1:10:12I'm going to tell the the next agent not

1:10:14to do something, never ends up actually

1:10:16being anything that you can trust. So in

1:10:20Grockbot and Dune and in Dune, uh we now

1:10:22have lint rules that give uh good and

1:10:25very helpful error messages telling the

1:10:27agent to fix the root cause, the real

1:10:30problem, uh rather than just patch

1:10:32around it.

1:10:34And so our general principle is that

1:10:36every time we correct our agents, it

1:10:37turns into one of these deterministic

1:10:39checks or we think about how to

1:10:41categorically eliminate that problem in

1:10:43the code through better API design,

1:10:45through better uh architecture. And if

1:10:48you do this enough times, you'll find

1:10:50that you have just less things to look

1:10:53out for in code review. And at some

1:10:55point, you'll realize there's there's

1:10:57nothing left to review, and you can just

1:10:59let agents merge their most of their own

1:11:01pull requests.

1:11:04All right. Uh, moving on. Verification

1:11:07is also a really big part of how we make

1:11:09our agents more autonomous and also

1:11:12increase our confidence and trust in

1:11:13their work. And our code review process

1:11:16has now really shrunk into looking at

1:11:18verification artifacts instead. So

1:11:20things like videos, screenshots,

1:11:22diagrams, formal verification.

1:11:26Um, and the most basic example of this

1:11:28is runtime verification. uh which is

1:11:30just a fancy way of saying giving your

1:11:32agent a way to run the code for itself

1:11:34and use debugging tools that you know

1:11:36human developers use as well like the

1:11:38Chrome DevTools protocol. Um in our

1:11:41runtime verification skills we have this

1:11:43thing that we call a feature map and

1:11:45that's basically kind of like a site map

1:11:48of all of the features that are

1:11:49available in your application from the

1:11:51user's point of view. And so with this

1:11:54agents don't need to guess when you say

1:11:56you know the thingy in the left sidebar.

1:11:58It gives you some shared vocabulary

1:12:01and we also make use of custom CLIs

1:12:03inside these skills so that uh agents

1:12:06can reuse the the CLI and the driver

1:12:09every time which cuts down on token

1:12:10usage and is also testable.

1:12:14Um and then recently we've also started

1:12:16looking at formal verification. Uh so we

1:12:19now use TA plus to do some model

1:12:21checking of major user flows such as uh

1:12:23notifications across multiple devices

1:12:25and platforms. Uh we use a lean as well

1:12:28for proving certain algorithms. Uh for

1:12:30example, we have our own we wrote our

1:12:32own custom virtualization library and we

1:12:34use lean there to check that we don't

1:12:36have you know flickering or jittering

1:12:38problems. Um and we're still

1:12:40experimenting with this approach.

1:12:42Overall it seems very promising. Uh

1:12:44we've already found quite a few bugs uh

1:12:46with formal verification.

1:12:49uh but it's certainly not a silver

1:12:50bullet and we're treating it as another

1:12:54tool, another layer in our agents uh

1:12:57environment.

1:12:59Uh and then this is a very fast video

1:13:01but uh another thing that we just

1:13:03started looking at is property based

1:13:05testing as well for UIs. Uh so we use

1:13:07this library called Bombadil. Um and

1:13:10this has also caught a few UI bugs for

1:13:12us already.

1:13:15Okay, so zooming out, I saw this tweet

1:13:17from Cindy and six months ago she she

1:13:20said that she was or she was talking

1:13:22about not being able to trust her agents

1:13:25because of subtle bugs that were getting

1:13:27merged.

1:13:29And really what was missing for her in

1:13:30being able to trust her agents was the

1:13:32verification process. And now 6 months

1:13:35later, she says that 70% of her daily

1:13:37tokens go into verification. And I'd say

1:13:40that that tracks with what I see as

1:13:42well, that most of my token usage goes

1:13:44to verification.

1:13:46And if you really look closely at our

1:13:47process, none of these things here are

1:13:50new or novel techniques that we've never

1:13:52had before.

1:13:54These are just good software engineering

1:13:56tools, but ones that might have been

1:13:58prohibitively expensive to use before as

1:14:01they were hard to understand or write or

1:14:03maintain. But agents drastically cut

1:14:06this cost down. And really, we're

1:14:08entering a golden age of software

1:14:10testing and verification because agents

1:14:12let us test everything to a much greater

1:14:14degree than used to be economical.

1:14:18And then formal verification can also

1:14:20help you better reason about how your

1:14:22product should work and whether your

1:14:23agents really understand your intent and

1:14:25goals. Uh I made this new skill called

1:14:28state chart uh in my plug-in PAC potato

1:14:31stack uh which helps you visualize state

1:14:33machines from uh from the user's point

1:14:36of view. Uh, and it's I find this much

1:14:38easier to review alongside the videos

1:14:40and screenshots that we provide. Um, and

1:14:43so that's been great. Sorry for the

1:14:45flashbang. Uh, okay. Now, let's look at

1:14:48what the promised land looks like. What

1:14:51do you get in return for all of this uh,

1:14:53you know, investment?

1:14:56So, on the graphbot team, our velocity

1:14:58is uh, taking off and rapidly

1:15:00increasing. And last month, our team

1:15:02shipped a ridiculous 11,000 pull

1:15:06requests to production, which is a wild

1:15:08number. And our pace of releasing the

1:15:12client has also increased. And we cut a

1:15:14new stable version about once a day.

1:15:18And then the core team is also shipping

1:15:20much more. And we also have uh a lot

1:15:22more casual or driveby contributors who

1:15:25don't always have a lot of context uh

1:15:28but they're also contributing. Um, and

1:15:30in October we had about 227 people who

1:15:34contributed at least one pull request.

1:15:38So despite this velocity accelerating,

1:15:40you might be thinking to yourself like,

1:15:41isn't this going to be chaos? You know,

1:15:43isn't this going to be a bunch of slop?

1:15:45But actually, the number of uh PRs that

1:15:47get reverted has actually gotten lower

1:15:49on a relative basis and is steady at

1:15:51about half a percent.

1:15:54But I think the really interesting thing

1:15:56about this shipping pace and this volume

1:15:58of pull requests is that the performance

1:16:02of Grathbot has stayed not just stable

1:16:04but has improved on multiple dimensions

1:16:06compared to when we launched. So startup

1:16:09typing, opening new chats, switching

1:16:12chats, all of this has gotten faster and

1:16:15keep in mind Grathbot is still an

1:16:17Electron app for now. Uh so there is a

1:16:20ceiling, right? But it's already much

1:16:22faster than many other agent UIs out

1:16:25there. And for an Electron app with this

1:16:29high velocity and this many people

1:16:31contributing, I'd say that's pretty

1:16:33good.

1:16:35So before you jump into building out a

1:16:37software factory and you know automating

1:16:39everything that you can, whenever you

1:16:41correct your agent, try and do it by

1:16:43improving your codebase first. you know,

1:16:45start with a lint rule to prevent new

1:16:47occurrences of anti-atterns and then

1:16:49clean up those uh old patterns as well

1:16:52and then make sure that your

1:16:53verification story is really strong.

1:16:56A lot of this work is a prerequisite to

1:16:58automation anyway. And if your inner

1:17:01coding loop uh produces slop, then

1:17:04automating that loop just produces more

1:17:06slop and you need to throw more and more

1:17:08and more tokens at the problem, which we

1:17:10all know is very expensive.

1:17:13And once you've made these investments

1:17:14into, you know, code into your codebase,

1:17:17static analysis, runtime informal

1:17:19verification, you'll start to find that

1:17:21many things become possible with these

1:17:24foundations and that you can really

1:17:26easily build your own factory on top of

1:17:28these foundations.

1:17:31And when you're ready, Grabbot is a

1:17:32great way to build that uh outer loop.

1:17:34We have tons of connectors, routines

1:17:36that can kick off cursor cloud agents

1:17:38and projects uh which let you

1:17:40orchestrate thousands of hundreds or

1:17:43even thousands of agents. And as a quick

1:17:45example, you know, whenever we cut a new

1:17:47release now, Grockbot will automatically

1:17:49spin up a new cursor project for us that

1:17:52will fuzz the entire application, the

1:17:54entire build, and it will find bugs uh

1:17:56by running the app and using it like a

1:17:58real user. And then it's going to open

1:18:01fixed PRs for each of those bugs that it

1:18:03finds and it's going to automerge it as

1:18:05uh crazy as that sounds. And that's

1:18:08because our code base is good enough

1:18:10that pull requests tend to just be good

1:18:12by default. And most product code that

1:18:14we land no longer really requires deep

1:18:16human review.

1:18:20So when you have solid foundations and

1:18:23good organic architecture, you can turn

1:18:25slop into gold and you can ship with

1:18:28high quality uh high velocity without

1:18:31compromising on quality. My name is

1:18:33Potato, also known as Lauren. Thanks you

1:18:36again for listening to my talk.
