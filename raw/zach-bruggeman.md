---
url: https://ai.engineer/code/2026/talks/zach-bruggeman
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

7:48:54second here.

7:48:56Any 49ers fans?

7:48:59>> Okay. You know, that's a better reaction

7:49:00I've gotten to other events this year

7:49:01when I've asked, you know, about their

7:49:03local sports team because I've gotten a

7:49:04lot of just like muted signs. I have to

7:49:05hit the, you know, well, wrong audience

7:49:07for that type of joke. But glad at least

7:49:09it can hit this time.

7:49:15Continue

7:49:29help.

7:49:46current slide.

7:49:51That works.

7:49:59>> Perfect.

7:50:01>> All right. Good. Well, thanks so much

7:50:03for waiting everyone. My name is Zach

7:50:05and I'm an engineer at RAMP. I'm just

7:50:07really excited to tell you all a bit

7:50:08about how Ramp has evolved its

7:50:10engineering culture this year through

7:50:11the help of our robotic friend Inspect.

7:50:14As a quick introduction, Inspect is

7:50:16Ramp's collaborative coding agent. About

7:50:18a year ago, we decided to take a bet on

7:50:20building our own coding agent with every

7:50:22session running and being fully

7:50:24multiplayer. Inspect can work alongside

7:50:26you and your team, whether that be an

7:50:28Inspect itself, Slack, Linear, or any of

7:50:31our of our integrations. And now it goes

7:50:34far beyond just writing the code with

7:50:36the built-in design primitives, review

7:50:38agents, and verification flows that let

7:50:40any builder go from prompt to shipping

7:50:43all inside inspect. Inspect is also

7:50:45connected with all of our integrations

7:50:46companywide. So everyone's automatically

7:50:48connected the to the data and tools that

7:50:50matter most, whether that be telemetry,

7:50:52databases, or any of the other tooling

7:50:54available.

7:50:57Through this, we now see around 75% of

7:50:59all of RAMP's code uh being being

7:51:02written and shipped by inspect.

7:51:05When we started sharing Inspect publicly

7:51:07at the start of the year, the technology

7:51:08itself was the more novel part. Spinning

7:51:10up sandboxes in under 5 seconds, wiring

7:51:13up all that internal context, having

7:51:15agents work together in a multiplayer

7:51:16setting. People had a lot of questions

7:51:18about the plumbing. But now with much

7:51:21broader adoption of these concepts and

7:51:22Inspect itself now being a product on

7:51:24the market, the technology isn't nearly

7:51:26the barrier that it once was. Instead, I

7:51:29want to talk about the other angle, the

7:51:31lessons we've learned around building a

7:51:32culture around Inspect and what you can

7:51:34take away to inspire the people and

7:51:36teams that you work with.

7:51:39Uh I'm not going to talk as much about

7:51:40those technical aspects of Inspect

7:51:42today, but if you are curious about

7:51:43that, you can read our blog posts or

7:51:45take a look at pragmatic engineers deep

7:51:47dive that they did with us a few months

7:51:48ago. You can also follow my boy Dylan

7:51:51Garcia on Twitter. He's on his social

7:51:53media grind, but he's also been posting

7:51:55a ton of really great inspect insights

7:51:57uh since we've launched it. So, shout

7:51:59out Dylan. Please give him a follow.

7:52:02One thing about working at RAMP is that

7:52:04it really makes you start thinking about

7:52:06everything through the lens of saving

7:52:08time and money. And although I think it

7:52:10is important to understand the systems

7:52:12that you work with and understand how

7:52:13your tooling works, if you're a multiund

7:52:16or multi-,000 person organization and

7:52:18everyone has to go through say 15

7:52:20minutes of setup, once you start

7:52:23multiplying that across an organization,

7:52:25this is days lost for what is very w.

7:52:29Then if you consider how fast the

7:52:31frontier moves here on best practices

7:52:33and new ways to set up an agent, you

7:52:35realize this setup cost isn't just one

7:52:38time. Not to mention these advancements

7:52:40won't always spread equally. So now you

7:52:43have an indeterminate amount of time and

7:52:45effort spent on this maintenance without

7:52:46an equal spread of knowledge leading to

7:52:48a disparate outcomes across an

7:52:50organization. While you can perceive

7:52:52this as an advantage on individual

7:52:54level, when you're trying to win, you

7:52:56need everyone on your agame.

7:52:58So this was our first lesson. Build

7:53:01tools that kill information silos. When

7:53:04we found patterns that worked well or

7:53:06tools that benefited everyone, we could

7:53:08push it to our entire organization

7:53:10immediately. If you want to spend time

7:53:12tweaking your own setup, that's fine.

7:53:14But inspect is always available as a

7:53:15very solid baseline. And what we found

7:53:18is that we've never needed to mandate

7:53:20usage of inspect because it existing as

7:53:22this baseline reduced the need for

7:53:24people to reach for their own setups all

7:53:26the time.

7:53:28Part of what I think makes this powerful

7:53:30as well is that the reality of this

7:53:32industry is that it's unlikely that you

7:53:35as an individual always know the

7:53:37absolute best way to work with agents

7:53:39across everything. Even if keeping up

7:53:41with this is your full-time job, we all

7:53:43miss things, hold biases, and have blind

7:53:46spots. So, part of the design of Inspect

7:53:48is that there's just one single Inspect

7:53:51for the whole company. When people

7:53:53figure out better ways to use it or have

7:53:55ideas for what would make it better for

7:53:56the work they do, we're able to bring it

7:53:59to the entire company. The best part of

7:54:01this is that for most changes, this is

7:54:03invisible to everyone else. You just

7:54:05keep using Inspect as you normally would

7:54:07and benefit from the improvements that

7:54:09everyone's making over time.

7:54:12And what we found is that we really

7:54:14don't have to run these formal sitdown

7:54:16here's how you use AI effectively

7:54:18training sessions for Inspect. With

7:54:20every inspect session being public

7:54:22within the company by default, it's

7:54:24actually easier to work in public than

7:54:26in private. As such, you default to

7:54:29working in public. And you can learn how

7:54:31to use it by watching how your teammates

7:54:33use it in the channels that you're

7:54:34already in. And your teammates can learn

7:54:37from you. You can literally ask inspect

7:54:39to go through their recent sessions and

7:54:41tell you about how they work, how they

7:54:43structure prompts, or how they break

7:54:44down gnarly problems.

7:54:47I I think this has not just been helpful

7:54:49for the builders at ramp, but also for

7:54:51the inspect team itself. We frequently

7:54:54see the way that people use inspect in

7:54:56ways that we didn't originally imagine.

7:54:59This then informs what we should build

7:55:00towards, whether that should be more

7:55:02native primitives for common patterns or

7:55:05where tools and data are missing. All in

7:55:07all, it creates this self-sustaining

7:55:09loop where we're able to build towards

7:55:11what makes Inspect the most successful

7:55:13by seeing what people are already

7:55:15achieving with Inspect.

7:55:18This feeds into the second lesson that

7:55:19we've learned. It's all about building

7:55:21trust. I think it's fair to say that

7:55:24adopting agentic driven coding has been

7:55:26a massive trust building exercise for

7:55:28every engineer this year. It's helped

7:55:30that the models have become more

7:55:32competent at writing high-quality code.

7:55:34But there's a large part of this that's

7:55:36just learning to trust what an agent

7:55:38builds.

7:55:39So much of building trust in tools comes

7:55:41from learning from other people. After

7:55:43all, that's why most of us are here,

7:55:44right? You wanted to listen to people

7:55:47you respect, or at least I hope you

7:55:49respect, uh, teach you about the things

7:55:51that they learned. But if we limit our

7:55:53corpus of knowledge to just those who

7:55:55find the perverse enjoyment in public

7:55:57speaking, we're still not getting the

7:55:59full picture. This democratization of

7:56:02knowledge sharing is just fascinating to

7:56:04me as a concept. Instead of grouping

7:56:06around a select few leaders, can we

7:56:08build systems that inherently share the

7:56:10knowledge of everyone? This is what

7:56:13Inspect has uh built towards.

7:56:16Every time someone ships something with

7:56:18Inspect, people can see it right there

7:56:20on the work you've done. The session

7:56:22itself is an easy to share. It's a

7:56:24living artifact of how that change came

7:56:26to be. When a poll request goes up,

7:56:28anyone reviewing it doesn't just see the

7:56:30400line diff dropped over the wall and

7:56:32hope for the best. They can see the

7:56:35exact conversation that you had, all the

7:56:38tools that Inspect used, before and

7:56:40after screenshots, or even a video

7:56:43recording of the verification agent

7:56:44clicking through the app in the browser

7:56:46to prove its work holds up. It makes it

7:56:49effortless to show not just what

7:56:50changed, but why you can trust it.

7:56:54It also just makes it easier to bring

7:56:56people you don't work with all the time

7:56:57into the fold. A fun moment on this one.

7:57:00It was back in February. One of our

7:57:02engineers, Andrew, reported a bug in

7:57:04Inspect. So, naturally, I reached for

7:57:07Inspect and asked to investigate. What's

7:57:09funny here is that although Inspect had

7:57:12access to data dog for a while, we'd

7:57:14only just finished instrumenting inspect

7:57:15itself. I mean, inspect was primarily

7:57:18built by front-end e- product engineers.

7:57:20So, tracing is a little bit of a foreign

7:57:22concept for us. But in any case, it was

7:57:24a great time for us to put it to the

7:57:26test for inspect. And inspect says

7:57:30the instrumentation was causing the

7:57:31error. Yes, the very instrumentation

7:57:33that was letting us debug this error was

7:57:35actually causing this error. So, Dylan

7:57:38and I were not exactly convinced by the

7:57:40robot here. And Andrew is just like,

7:57:43whatever, I'll use cloud code. And and

7:57:45while we don't have a mandate to use

7:57:47inspect, I mean, personally, these were

7:57:50not the words I wanted to hear.

7:57:52So, nobody here is really trusting

7:57:55anybody. Dylan and I don't trust the

7:57:58robots explanation. Andrew doesn't trust

7:58:00us to build something that works. And

7:58:02inspect, well, I I I don't know if

7:58:05Inspect trusts anyone, but that's okay.

7:58:06That's okay. But because this is

7:58:09happening together in public, we keep

7:58:12proddding at it. Dylan asks inspect to

7:58:14look through the code of the telemetry

7:58:16library using and see if this was just

7:58:18something that was happening before

7:58:19without visibility because surely by now

7:58:22coding has been solved and there can't

7:58:23be bugs that we introduced and this time

7:58:26it comes back with something a bit more

7:58:27specific. The hotel library calls fetch

7:58:30but it never reads or cancels the

7:58:32response body. This was just a

7:58:33cloudflare platform limitation at the

7:58:35time albeit happening in a library

7:58:37specifically made for cloudflare workers

7:58:39but that happens. This library was

7:58:41written pre-agi after all. At this

7:58:44point, we ask inspect to make a patch.

7:58:46It updates the exporter to explicitly

7:58:48cancel the response body. We grumbled in

7:58:51the thread a bit while it's deploying.

7:58:53But we see after it deployed, there

7:58:54haven't been any errors since. So great,

7:58:57inspect fixed the bug, which at this

7:58:59point it's nothing special in itself.

7:59:01But there was a much more important

7:59:03story playing out in this thread.

7:59:06Before inspect, a report like this might

7:59:08have been acknowledged. a ticket filed

7:59:10and then one of us would have gone away

7:59:12to debug it. Andrew might have received

7:59:14a message later saying it was fixed, but

7:59:16the interesting work would have been

7:59:17done in private. Now though, it's easier

7:59:20to do all of this in public.

7:59:23And as such, Andrew naturally starts

7:59:25becoming engaged, wanting to chime in

7:59:27with his thoughts and experiences. And

7:59:29then in that same thread, this leads to

7:59:31ideas coming up about how Inspect could

7:59:33power other work. And none of this

7:59:35required any proddding or pressure to

7:59:37work together. instead it was just like

7:59:39oh these guys are doing something cool

7:59:41and we could work on it together.

7:59:45So in the span of an hour we go from a

7:59:47skeptical user with a broken user

7:59:49experience to an engaged colleague

7:59:51wanting to build together and a more

7:59:53stable inspect.

7:59:55So once you have a tool that gives

7:59:57everyone the best setup on day one with

7:59:59no information silos and you've built a

8:00:02system of trust that brings innovation

8:00:04together, what happens next? This is the

8:00:07last lesson I want to share. I think the

8:00:09only thing left is to think bigger

8:00:11together.

8:00:13Something that's pained me this year uh

8:00:15is how much reverence has been built for

8:00:16the lone wolf operator. It's great that

8:00:19we can all do more individually with the

8:00:21help of AI, but this can't be the

8:00:23endgame in my opinion. If we don't build

8:00:26outwards, collaborate with other people,

8:00:29and continue to share the progress and

8:00:31learnings we make, I think this is all

8:00:33going to collapse in on itself. We'll

8:00:35only consider ourselves as our target

8:00:37audience and we won't grow through the

8:00:39consideration of others perspectives and

8:00:41experiences.

8:00:42This is not to say I'm all doom and

8:00:44gloom about this. The fact we're all

8:00:45here hopefully is a testament to the

8:00:47value of connection. More importantly

8:00:49though, I think we can shape our tools

8:00:51to enable these connections to flourish

8:00:53even stronger than before.

8:00:56That story I told about Andrew has

8:00:58played out so many times within ramp.

8:01:00It's probably the most inspiring thing

8:01:01about Inspect to me, honestly. Sure,

8:01:04it's a coding agent, but remarkably,

8:01:06it's become part of the culture.

8:01:08Everyone at RAMP knows its face. We

8:01:10dress it up on holidays. We throw in

8:01:13Easter eggs over time. And we evolve it

8:01:15as RMP itself has evolved. Not all

8:01:18iterations have been perfect, as any

8:01:20RAMP engineer could tell you. But I do

8:01:22think every iteration has been together.

8:01:26Going back to earlier this year, Inspect

8:01:28didn't have nearly the amount of

8:01:29functionality it has today. A great

8:01:31example is its code review agent. Before

8:01:34this, we were using various

8:01:36off-the-shelf code review products,

8:01:38frequently rotating between them to find

8:01:39a good fit. At some point though, a

8:01:42couple engineers, Daniel and Franel,

8:01:43realized that we could probably build it

8:01:45ourselves as Inspect already had

8:01:48automation triggers that fired from a

8:01:49variety of sources, including when a

8:01:52pull request was opened. With just what

8:01:54we built already, they got to work

8:01:56building another beloved agent at RAMP,

8:01:58review buddy. And what was so cool about

8:02:00this is that much like Inspec's own

8:02:02origins as a side project, these

8:02:05engineers were working on this. They

8:02:07didn't need to ask for permission to

8:02:08build it. They just challenged

8:02:10themselves to build something better,

8:02:12used inspect to move faster, and quickly

8:02:14built an agent that was better for all

8:02:16of ramp.

8:02:18This really did force us working on

8:02:20Inspect to think about Inspect

8:02:22differently. At that point in time, we

8:02:24were pretty okay with Inspect quote

8:02:26unquote just being a coding agent. But

8:02:29once we saw what others were pushing it

8:02:31to do, it got us thinking about how

8:02:34Inspect could become full circle. If

8:02:36this push didn't happen naturally, I

8:02:38think it would have taken the Inspect

8:02:40team a lot longer to get there.

8:02:44This is what I mean about thinking

8:02:45bigger together. Especially with the

8:02:48continued improvements in model

8:02:49performance, now is the time to push

8:02:51yourself and the people around you to

8:02:53think bigger. I encourage all of you to

8:02:56take some time in this coming week to

8:02:58ask everyone that you work with, what

8:03:00have we been waiting to do that we

8:03:02haven't done yet. And once you found

8:03:04those things, try to build them

8:03:06together. There's really not an excuse

8:03:08to not just try anymore. You have access

8:03:11to the same models as everyone else. But

8:03:13you have to start somewhere. And I

8:03:15believe that there is so much potential

8:03:18in unlocking wins, not just for

8:03:19yourself, but for the people around you.

8:03:22Inspect itself was born this way. not

8:03:24only as something that I wanted to build

8:03:26and Jason wanted to build, but out of a

8:03:28want to build something for a team that

8:03:30I truly love and truly respect that

8:03:33would just make their jobs a little more

8:03:35enjoyable.

8:03:36Now, if you'd like to use a coding agent

8:03:38that has embodied all these lessons that

8:03:40we've learned, I'm happy to say that

8:03:41Inspect is now available outside of

8:03:43RAMP. If you go to inspect.com/ieie,

8:03:47we'll prioritize you and your team on

8:03:48the wait list. We're actually moving

8:03:50through this weight list pretty quickly,

8:03:51so expect to hear from us soon. But

8:03:53otherwise, I hope you've enjoyed hearing

8:03:55about our robotic friend, Inspect. If

8:03:57these topics interest you, feel free to

8:03:58reach out here or an X. But otherwise,

8:04:01thanks so much for listening.
