---
url: https://ai.engineer/code/2026/talks/zach-lloyd
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

1:01:22All right. Hi, everyone. Excited to be here. And today I'm going to chat about the evolution

1:01:28of software engineering and how it's becoming factory engineering. So excited to do this.

1:01:35First, a little bit about me. I've been an engineer for a long time. I was a principal

1:01:40engineer at Google. I used to lead engineering on the Google Docs suite, helped scale that

1:01:46product to hundreds of millions of users.

1:01:48I'm currently the founder and CEO of Warp.

1:01:51Warp builds a platform for companies to build their own

1:01:56internal software factories.

1:01:57I'm going to show a little bit of that today.

1:02:00Warp is also well known for our open source agentic terminal.

1:02:05It has about 65,000 GitHub stars.

1:02:07It's used by almost a million developers.

1:02:10Our mission has always been helping developers ship better

1:02:13software more quickly and it's cool to evolve what we're doing as the world changes around

1:02:18us.

1:02:20is where I think we're still mostly at.

1:02:24Most developers that I talk to, most platform teams, they're working with individuals who

1:02:31are running their own coding agents locally.

1:02:34Maybe they're running multiple of them at a time.

1:02:38It could be through their terminal.

1:02:39It could be through an agentic development environment.

1:02:43Just a show of hands quickly, actually.

1:02:45How many people in here are mostly working in this fashion?

1:02:48Local, multi-threading.

1:02:51Okay, so still most people, but actually not everyone.

1:02:56I think that this is going to change,

1:02:57and it's starting to change already.

1:03:01The reasons are, there's a few of them.

1:03:03So one is, like, if you are doing local interactive prompting

1:03:07is your main way of working with coding agents.

1:03:10You're kind of limited by human throughput, really.

1:03:14You cannot automate if every agent needs

1:03:18to be interactively prompted.

1:03:19There's a second reason, too, that I hear a lot

1:03:22from platform teams and from engineering leaders.

1:03:26This mechanism of everyone choosing their own local setup,

1:03:30their own models, their own skills,

1:03:33makes it very hard to actually make sure

1:03:35that you're using these agents efficiently from a cost perspective and a productivity

1:03:40perspective. And then finally, there's a big security problem and governance problem when

1:03:46everyone is setting up and running their own local agents. And so the thing that is coming,

1:03:52I loosely call a software factory. That means different things to different people.

1:03:59I'm curious for people in this room, how many folks are currently trying to build or building

1:04:05or using their own internal software factories?

1:04:07Show of hands.

1:04:09Okay, so I would say that's almost more

1:04:12than the interactive agents at this point.

1:04:13That's super cool.

1:04:16So I'm gonna define what I think of as a software factory,

1:04:20but I'll start by saying what I think

1:04:21a software factory is not.

1:04:24I think it's not just a loose set of automations.

1:04:29I don't think of a software factory

1:04:30as a collection of AI teammates.

1:04:34I don't think it's a shared set of skills and MCPs, and I also don't think software

1:04:38factories are personal agent swarms.

1:04:40I think these are all useful primitives, but they're not the end state that we should be

1:04:47shooting for.

1:04:48And so to me, the way that I define software factory is very specific, and I'll unpack

1:04:55this, but I define it as a code-defined closed-loop system for continuously measuring and improving

1:05:02cloud coding agents.

1:05:04So that's a mouthful, but let me unpack it.

1:05:06So if you want to build a software factory and really scale it, I believe the first thing

1:05:12you need to do is fully encode it in a declarative language.

1:05:17So you need to say, okay, these are all of the coding agents that I want to run.

1:05:22These are how they're triggered.

1:05:24When work comes into this factory, this is the workflow from triage to spec, whatever

1:05:30your workflow is.

1:05:31you need to specify what MCPs are available,

1:05:34where your agents run,

1:05:36really as much as you can

1:05:38in a Terraform-style declarative, you know,

1:05:42set of code files that define your factory.

1:05:44And I'll talk about why this matters in a bit.

1:05:47The second part of it

1:05:48is a continuous closed loop of improvement.

1:05:51And that closed loop can look something like this.

1:05:54This is the typical one.

1:05:56This is just really the SDLC,

1:05:58where you have ideas that come in from people

1:06:02They could come in from automated signals

1:06:04And then your agents do some amount of work

1:06:08Taking that issue through various stages

1:06:11From triage, implementation, verification, monitoring, and so on

1:06:15The important piece here is really this closed loop concept

1:06:18Where the more that your agents run

1:06:21The more efficient you can make this loop

1:06:24And I'll talk about how you do that in a second

1:06:27The third really key aspect

1:06:29if you're building a software factory

1:06:31is that everything needs to be measurable.

1:06:34If you're just going on vibes

1:06:36in terms of how you deploy coding agents,

1:06:39you can't be confident that you're actually

1:06:41getting the best, highest quality outputs

1:06:45at the best cost.

1:06:46You can't be confident you're maximizing throughput.

1:06:48So you really need a measurement mindset

1:06:50and measurement capabilities.

1:06:53so if there's one thing

1:06:55that I'm hoping people walk away from this talk with,

1:06:59it's the idea that the way to work with coding agents

1:07:03and the way to think about them

1:07:04is that there's a really cool fundamental new engineering problem,

1:07:09which is how do you take this magic new primitive

1:07:13that we all have, that we've got in the last year,

1:07:16that you have these intelligent agents,

1:07:18and how do you engineer a system around them

1:07:21to use them most effectively?

1:07:23To me, it's not about growing a garden

1:07:26or following a recipe.

1:07:29we're all engineers.

1:07:30We've spent our careers

1:07:32learning how to solve problems from an engineering mindset.

1:07:35And when you think of coding agents

1:07:36as a new engineering primitive,

1:07:39then you naturally want to,

1:07:41you'll naturally start to build

1:07:43a certain way of using them.

1:07:45That's the mindset.

1:07:47So yeah, just to unpack that a bit,

1:07:49so there's continual measurement

1:07:51and continual learning,

1:07:53but there's also a new set of tools.

1:07:56And so when I was learning how to become a better engineer at Google, you know, I really

1:08:02had to get good at using things like debuggers and profilers and Chrome DevTools.

1:08:07And basically, there were all of these tools of the trade for building software by hand

1:08:12that maybe are not as important now, but they really separated the best engineers from the

1:08:19okay engineers.

1:08:20And in the new world of factory engineering, there's a new set of tools that everyone should

1:08:26be trying to master.

1:08:27And so those are things like evals.

1:08:30It's can you set up self-improvement loops that work?

1:08:34Can you write benchmarks?

1:08:36Can you A-B test your factory?

1:08:39And that's a really important skill set.

1:08:42And mastering that skill set is something that I think of as factory engineering.

1:08:48Now, that doesn't mean that that's the only kind of engineering you're going to be doing.

1:08:53You're still going to be doing product engineering, meaning you as an engineer should still feel

1:08:59responsible for, you know, does the product you ship actually solve a real user or customer

1:09:05problem?

1:09:06But the best engineers, and I would say like every engineer really, is going to need to

1:09:11learn this new tool set of how do you improve the system, not just build the product.

1:09:17And so what does that look like?

1:09:20If you think from first principles about how do you engineer an agentic system, I think

1:09:25you come up with something like this.

1:09:27It doesn't have to be exactly this, but it's kind of an infrastructure stack that you're

1:09:32going to want to use to engineer your agents.

1:09:34And so at the bottom of this is what I call factories is code.

1:09:38I talked about this earlier.

1:09:40It's defining all of your coding agent workflows in a code format.

1:09:45And the value of that is that you get the ability to branch off it and test different factory configurations

1:09:51you can roll back and

1:09:54then crucially it opens up the way for self-improvement

1:09:57Which is the ability for coding

1:10:00agents themselves

1:10:00to suggest changes to how your factory works to get that continual loop going.

1:10:07Above that is a layer of data and context. This is also crucial. So if you are running

1:10:13coding agents at scale in an organization, you really must be recording everything that

1:10:20those agents are producing. So all of their conversations, all of their artifacts, the

1:10:24videos that they make, the screenshots. And the reason for doing that is that's the raw

1:10:29material that you will use to improve the way your factory works over time. On top of

1:10:36that is a compute layer. So you need to have sandboxing. Sandboxing increases security.

1:10:42It makes your factory more repeatable. Your sandboxing should be probably multi-platform.

1:10:47On top of that is the inference layer. The way that I would think about inference and

1:10:52models in a factory is they are a degree of freedom to test.

1:10:57And so your factory, however you build it, should support multiple models.

1:11:03It should also support multiple coding agents because, you know, Cloud Code might be better

1:11:08for some things.

1:11:09Codex, Pi, Warp, these harnesses all have different properties.

1:11:13You might want to write your own coding agent harness that has some of your organizational

1:11:17context baked in.

1:11:19So your platform should support that.

1:11:21On top of that, you need an orchestration layer.

1:11:23So this is what actually moves work through the factory.

1:11:26In some ways, the least interesting part, but you need that.

1:11:29And then finally, your factory should be all API and MCP first.

1:11:34So that it can be used not just by, you know, people, but it can be used by other coding

1:11:39agents.

1:11:40And it can go to where the work is being done.

1:11:43So your factory should integrate into things like Slack, Linear, Jira, GitHub, and so on.

1:11:50I'm going to give a quick demo of what some of these tools look like in the context of

1:11:55warp factories, but this stuff I'm going to show I think applies also if you're building

1:12:01your own.

1:12:04So let me start here.

1:12:06So at the foundation I mentioned earlier is factories in code.

1:12:12And so I'm going to show some code.

1:12:14It's not going to be that interesting, but if you're familiar with something like Terraform,

1:12:18you will get what's going on here.

1:12:20You basically want a declarative way of saying,

1:12:24this is my factory.

1:12:25My factory has certain policies.

1:12:28It accesses certain repos.

1:12:30It has certain triggers.

1:12:32It has a sort of blueprint that defines the workflow

1:12:35of work going through it.

1:12:37And then if you zoom out,

1:12:39all of these other things that are part of the factory

1:12:41also have code definitions.

1:12:44The second thing, moving up a layer,

1:12:47is the idea that everything that you do in the factory

1:12:58should be tracked across your entire team.

1:13:03And so think of what I'm showing here.

1:13:06This is a sort of Google Docs style live view

1:13:09of what every coding agent on Warp's team

1:13:11across all of our engineers are doing at all times.

1:13:15And so I could click it and I could see,

1:13:17okay, I don't know what Maggie is working on,

1:13:19but I know I could look at it.

1:13:21She's implementing Opus 5.5 on the high plan.

1:13:23She's doing it from Slack.

1:13:25It has this artifact.

1:13:26I could look in and see the actual conversation here.

1:13:29I could see what it's generated.

1:13:31And so I could see this in real time, which is cool.

1:13:34But the really important thing

1:13:35is that you gather all this information

1:13:37so that it can be used for improving the factory.

1:13:42And so that brings me to the third thing,

1:13:45moving up the stack here.

1:13:46is once you have all of this information

1:13:48that's being stored centrally,

1:13:51you get the ability very easily,

1:13:53without doing any, like, install stuff

1:13:56on everybody's laptop, anything like that,

1:13:58to understand how these coding agents are working.

1:14:02And you can think of that in terms of, like,

1:14:04traditional output metrics, like your DORA metrics,

1:14:07like, okay, cycle time.

1:14:09But you can also think of that in terms of, like,

1:14:11how automated am I, like, how many human interactions per PR,

1:14:15what's your throughput.

1:14:16you should be able to see data very clearly here

1:14:20on, like, what's your cost.

1:14:22Again, surfacing this, if you don't have this data,

1:14:26you're kind of flying, uh-oh.

1:14:30You're kind of flying blind.

1:14:32So this is really important.

1:14:33And then there's this next layer of data,

1:14:37which is also really important that I mentioned earlier,

1:14:39which is the ability to set up online evals

1:14:42that run across all your coding agent sessions.

1:14:44And so what I mean by this, an eval is just like,

1:14:48as these coding agent sessions are running

1:14:50and they're recording their output,

1:14:52you want the ability to go back and have another agent

1:14:56look at them and score them on some dimension

1:15:00that might indicate how well they're doing.

1:15:02And to make that more concrete,

1:15:04so it's like we have an eval, a score here that we use

1:15:08that is looking for redundant tests.

1:15:10This is a good example.

1:15:11And so one common failure mode of coding agents

1:15:14is that sometimes they will write tests that are unnecessary.

1:15:17We want to catch that, and that's what this eval does.

1:15:21We want the ability then to see,

1:15:23okay, when it's writing too many tests,

1:15:26why is it doing that?

1:15:28And is there a mechanism that we could change

1:15:31in the factory for improving that?

1:15:33And so, again, just to kind of recap here,

1:15:36you want a way of recording all the runs.

1:15:40When these runs run, I didn't show this,

1:15:42but all of the runs are, you know, you want a way of specifying

1:15:49what is the sort of, like, actual loop that they're running through.

1:15:52You want to be able to see where they are in that loop.

1:15:55As the runs work, you want to be able to gather their data.

1:15:59Once you gather their data, you want to be able to set up these online evals.

1:16:03And then these online evals give you a sense of, like,

1:16:05okay, where are there opportunities for improvement?

1:16:08Once you have that in place,

1:16:10You can do something that we call self-improvement

1:16:12Which is rather than having humans

1:16:15Look at the outputs of these scorers

1:16:16You have agents look at the output

1:16:18And so the way to think about self-improvement

1:16:21Is like it looks at the score output

1:16:24It says okay we saw some number of failed runs

1:16:30Let's try to look for patterns

1:16:32In those failures

1:16:33And suggest a possible solution

1:16:37And then the form that that solution takes is as a PR against your factory definition.

1:16:46And so this is like the awesome thing about having your factory defined as code and being

1:16:51able to use coding agents is that you can then have the coding agents diagnose problems

1:16:57with how they're working internally and suggest fixes to them.

1:17:01The final tool that I'm going to point out that I mentioned earlier is the ability to

1:17:06create and run benchmarks. This is also essential. So if you're following this course online,

1:17:13you'll often see things like, okay, this is the best model, this is the new best skill,

1:17:16this is the best harness. The only way to answer those questions for real is to test

1:17:22on your own data. And so you should have in place some mechanism where you can basically

1:17:27create a benchmark set out of your real data

1:17:31and then replay those tasks.

1:17:35Oops, I'm on the wrong one here.

1:17:37And replay those tasks against different configurations

1:17:41to see how they would have done

1:17:44if you had varied some model or varied some harness

1:17:47or varied your factory configuration.

1:17:49So just to make this clear,

1:17:54this is something that looks like what you might get when a new model drops like in sweet bench or

1:18:02terminal bench it's showing the trade-offs of different models here in terms of cost and

1:18:08quality you want to be in this upper left-hand quadrant but it's on your own actual data and you

1:18:14can you can test not just models but also you want to be able to test hey if i got rid of these skills

1:18:18is does my factory perform better or worse?

1:18:20So these are some of the tools.

1:18:24And just to return to this big concept,

1:18:27like this has worked well for us.

1:18:28We have managed to use these tools

1:18:31to drastically improve our throughput

1:18:33and lower our cost.

1:18:35We are just constantly doing this

1:18:38because this cost really matters to us,

1:18:40where we're getting a performance baseline,

1:18:42we're scoring, we're benchmarking, we're testing,

1:18:45and we're doing factory engineering.

1:18:47And so that brings me back to this last point

1:18:50that I just want to emphasize again.

1:18:53Engineer your agents.

1:18:54It's a new engineering primitive.

1:18:56We're all engineers.

1:18:57Think about how you can get the most out of them

1:18:59and build the system for that.

1:19:01I think this is very exciting.

1:19:04I know there's a lot of, like, angst

1:19:07around what it means to be an engineer right now.

1:19:09I know we're not writing code,

1:19:11but I'm still using all the engineering skills

1:19:13that I spent my whole career learning,

1:19:15And so to me it's still a really exciting time

1:19:18If you're interested in more factories

1:19:19You can check it out here

1:19:21Thank you all for coming to the talk
