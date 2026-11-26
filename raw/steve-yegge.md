---
url: https://ai.engineer/code/2026/talks/steve-yegge
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

8:06:31Hey everybody.

8:06:33All right, exciting times. Thank you.

8:06:35The room is really full. That's awesome.

8:06:37Thank you all for coming at the end of

8:06:39the day. Uh I'm really excited to uh be

8:06:42here and to chat with you. I write a a

8:06:45you a a custom speech every single time.

8:06:47So this is this speech was written just

8:06:50for you today based on talking to a

8:06:53bunch of you. Uh I've been in the

8:06:55industry for about 40 years. I worked at

8:06:58Amazon and Google for about 20 years.

8:07:01I've been all over the place, done all

8:07:03kind of stuff. And uh and uh for the

8:07:06last year or a little bit more I've been

8:07:08independent and just kind of like

8:07:09playing with um playing with software

8:07:12factories. Yeah. So um I kind of

8:07:15invented them. Right. On January 1st I

8:07:18launched Gas Town. I don't even think we

8:07:19called them software factories back

8:07:21then. I think I called it an

8:07:22orchestrator. But I knew that I wanted

8:07:24Claude to run Claude. Right. And it

8:07:27turned out to be a little trickier than

8:07:29than I thought. I spent all last year on

8:07:31it and I wrote

8:07:33That's interesting. So I can't see my

8:07:35slides, but you can see my slides. So

8:07:37that'll work. So I wrote two

8:07:38orchestrators last year. They were an

8:07:41attempt to make the LLM smarter. Vibe

8:07:45coder and VC, you know, to do

8:07:47engineering stuff like, you know, do

8:07:50code reviews and all that stuff, right?

8:07:52Really, really bad idea. Just complete

8:07:55failure. I just built these huge

8:07:56monoliths. I was fighting the bitter

8:07:58lesson. Don't try to make LLM smarter.

8:08:00They will get smarter. And after a

8:08:03couple of months of this, although beads

8:08:05did fall out of it and beads was a

8:08:06really cool discovery and beads, beads

8:08:09is is life. Who here uses beads?

8:08:12A few. Yeah, pretty good. Actually,

8:08:14pretty good. Yeah, we'll talk a little

8:08:16bit about it. Excuse [clears throat] me.

8:08:18And then I had Gas Town. And then I had

8:08:20in June I had Wheelhouse, which was my

8:08:23own. So I decided to build a factory

8:08:25from scratch just to work on my video

8:08:27game, right? Because like everyone's

8:08:30building factories to build factories.

8:08:32It's kind of naval gazing. It's kind of

8:08:34like Yeah. Right. You know, it's like

8:08:35why they're not, you know, I think more

8:08:38people need to build stuff, not just

8:08:41tools for building stuff, right? And I I

8:08:43this is going to be a theme today,

8:08:44right? And and the thing is the one

8:08:47pattern that I've seen here is all of

8:08:49them go through a phase where the models

8:08:52get really excited and they build

8:08:53something really cool for you and then

8:08:56it just turns into garbage mountain

8:08:59every single time. It happened to Gas

8:09:01Town. It happened to Willhouse several

8:09:03times. I had to burn the whole thing to

8:09:05the ground a couple times. Right. It's

8:09:08almost like like every model family,

8:09:10every model generation comes in and can

8:09:13build something that's just a little too

8:09:14big for it to handle. And then the next

8:09:16model generation comes in and goes,

8:09:18"Yeah, we're going to have to burn

8:09:20that." Right? And I It's a pattern. I've

8:09:22done this over and over again. And I can

8:09:24tell you some things about it. But

8:09:26first, let me tell you about my game.

8:09:27It's a fun little RPG. I've been working

8:09:30on it for 30 years. It has players that

8:09:33started in 2001. So, I literally have

8:09:35players that have been playing for 25

8:09:37years. They're still playing. Like, you

8:09:39can log in right now and find probably

8:09:4010 people just screwing around doing

8:09:42quests, live quests, and and whatever.

8:09:45It's a It's a player uh you know, the

8:09:48players contribute the content, right?

8:09:49It's a MUD. Like, they were really big

8:09:51in the 90s. They're like text games, and

8:09:53this one has graphics on top of it,

8:09:54which turns out to be like really ideal

8:09:57for AI agents because it's text. They

8:09:59can literally play the game via TNET,

8:10:02and they asked me to. Uh, so we'll talk

8:10:04about that too. Here's an older

8:10:06architecture diagram of what my sort of

8:10:09of what my system looks like. So when I

8:10:12built this wheelhouse software factory

8:10:15for Wyvern without a care in the world

8:10:18for making it, you know, usable by you,

8:10:22right? No open sourcing, just bespoke.

8:10:27I discovered the alien face hugger

8:10:29pattern. All right, which is where your

8:10:32factory turns into an alien face hugger

8:10:34and like gloms onto your product and

8:10:37just becomes

8:10:40grown into it. Okay, like look, we could

8:10:44turn off the AI and still run the game

8:10:46today. Me and the admins and the players

8:10:49and the community and we could we could

8:10:51but we would miss the AI at this point.

8:10:54The players are talking to my factory.

8:10:57The factory is talking to my admins.

8:10:59They're asking for features. I'm not

8:11:01even in the loop. It's like a wish

8:11:02factory. They're building the game now.

8:11:04They're all old-timers and there's a set

8:11:06of them that I trust to make, you know,

8:11:08wise decisions about the game. But

8:11:10they're they're all working with my

8:11:12factory now. And they'd be mad. The

8:11:14funny thing is when it first started,

8:11:15they were like, "Time out. What is this

8:11:18AI stuff for starters?" And they were

8:11:21like, "No AI art." Because they could

8:11:23see it was slop. I'm like, "Okay, we got

8:11:25human artists." And they said, 'But we

8:11:27do kind of like all the new features,

8:11:29but we're really scared of the pace. Can

8:11:31you give us a road map? My players came

8:11:34to me asking for a road map. They were

8:11:36like, 'You got to slow down. It's too

8:11:37much. [snorts] And I was like, okay. And

8:11:39I took a look and my software factory,

8:11:42this thing had been cranking out like 50

8:11:45to 100 features a day. I had a hundred

8:11:48years of work in my backlog when I

8:11:49started this and Claude got through

8:11:50about 60 years of it in about four

8:11:52months, right? It's absolute insanity.

8:11:54and they can't look at a 100 features a

8:11:57day. They can't even like read about 100

8:11:59features a day, let alone play test

8:12:01them. I was my factory was completely

8:12:03overwhelming my customer base, my users.

8:12:07It was also melting my machines and it

8:12:09was like annoying my accountant.

8:12:13I mean, she likes working with BAT as my

8:12:16head of finance. Uh, but she sent me a

8:12:18$1,200 bill because, right, the thing

8:12:21sent her 20 mails. This is my factory.

8:12:23This is what it looks like. It's so

8:12:25beautiful. Look how beautiful it is. Um,

8:12:27okay. I admit it's a little ugly. Oh,

8:12:29and this is a really old screenshot of

8:12:30it, but whatever. It's basically, it's

8:12:32an Emacs

8:12:34because I don't care about you.

8:12:37Uh, there are people who are building

8:12:40stuff like this like Superlogical from

8:12:42Mitchell Hashimoto and I don't know,

8:12:44maybe the Herder folks of the world, you

8:12:46know, for running like lots of agents,

8:12:48but this is like 50 agents. Okay, I'm

8:12:52actually like operating in the future.

8:12:55Okay, and I'll tell you how I do it. I

8:12:57have 22 Cloud Max accounts.

8:13:02All right, and thank you to Anthropic

8:13:05for letting me continue this. Uh Jeffrey

8:13:07Emanuel has 28. I actually had [snorts]

8:13:09to stop when they added the phone number

8:13:10verification. I didn't want to add

8:13:11anymore. Anyway, I asked my factory at

8:13:13one point. I said, "Hey, hey, how much?"

8:13:15Because they kept going, "Hey, give us a

8:13:16couple more accounts. Couple more.

8:13:17Couple more. Couple more." It's like 200

8:13:19a month, right? And I'm just like, so I

8:13:20finally said, "How many do you want?"

8:13:21They're like, "Five, seven more." And

8:13:23I'm like, "How many do we need steady

8:13:25state to keep this factory operating

8:13:27full throttle?" And they said

8:13:3055 accounts. I was like, "No, no, I have

8:13:3420. It's already like 20 Chrome profiles

8:13:36that I got to log into and no." So I

8:13:38told him, "You're we're going to start

8:13:40focusing on fuel efficiency."

8:13:42And so and I'll talk about that. Right?

8:13:44So the these are the five horsemen of

8:13:46this is what I've had to deal with this

8:13:48year. I I'm telling you I'm operating in

8:13:51the future because like you're all going

8:13:52to be here in 12 to 18 months when

8:13:55either inference costs drop

8:13:57significantly which may or may not

8:13:59happen or distillation causes fable

8:14:02class uh intelligence to be super

8:14:05readily available and you don't have to

8:14:06have a lovely wife who lets you spend

8:14:09$5,000 a month out of pocket just on

8:14:11inference and another 5,000 on cloud

8:14:13compute because my game ain't making

8:14:1510,000 a month right now for my old

8:14:17players, right?

8:14:18uh but we're doing it because we're

8:14:20learning and um and this is this is what

8:14:23this is what I I have to deal with. So

8:14:26first first we'll start about with work.

8:14:28All right, you need a way to represent

8:14:30the work that's going to happen in your

8:14:31system and when you're building a

8:14:33software factory it's all about pumping

8:14:35lots of work through. Okay, it really

8:14:38becomes like a work is like this fluid

8:14:40that you have to like it gets stalled up

8:14:41in pools and oh they're all waiting on

8:14:43security desk. You're like this is

8:14:45pluribus man. You're all the same model.

8:14:47Why does it have to wait on security

8:14:48desk? You spend a lot of your time

8:14:49arguing with your factory. [snorts] It's

8:14:52uh it's complicated inter intelligence

8:14:54tiering. You have to tar your work. You

8:14:56have to route your work. You know, there

8:14:58aren't any good solutions for this today

8:15:00except beads. As far as I can tell,

8:15:02beads is really, really, really good for

8:15:04this. It's a agent speed task tracker

8:15:08that turns into a knowledge graph

8:15:09automatically for you. And then as

8:15:12you're tracking work, this is the kind

8:15:14of stuff I have to do every morning.

8:15:16Every morning with my factory, I'm like,

8:15:18"Oh, looks like we had a stall. Four

8:15:20four workers didn't do anything last

8:15:22night. Oh, and and these the these these

8:15:25ones went way too fast, and we have to

8:15:26slow down and wait for human gates." You

8:15:29know, what are you guys even doing?

8:15:30Observability is always really hard. Uh,

8:15:33it's, you know, management of work

8:15:35becomes just this this thing that you

8:15:37got to do. And the next thing you got to

8:15:39deal with is fuel

8:15:41tokens.

8:15:43It's just so painful, I'm telling you. I

8:15:46mean, I'm spending 5,000 a month and

8:15:49every one of those tokens is like

8:15:50precious to me, right? Because it's out

8:15:52of my own pocket. I think I'm spending

8:15:54about five billion a day of tokens. And

8:15:57it would be the the API spend equivalent

8:15:59if I were on an actual API tap would be

8:16:02about one and a half million a year. At

8:16:05full throttle, it would be three million

8:16:06a year. So again, I'm doing something

8:16:09that most people can't do. You can't do

8:16:12this at a company because it would be

8:16:13prohibitively expensive. Like the PLG,

8:16:16you know, the the uh the $200 accounts

8:16:19for Codeex and Cloud Max, that's their

8:16:21Costco hot dog. That's their loss

8:16:22leader, right? And and and that was TK,

8:16:26by the way, that came up with that

8:16:27analogy. He might be here. And uh and

8:16:29so, you know, I'm hopeful that that

8:16:31they're going to let us keep keep

8:16:32innovating because I think that I've

8:16:33gotten some pretty good training

8:16:34sessions for for Claude out of this. So,

8:16:36we all benefit. But I tell you, every

8:16:39problem is a fuel problem. You want your

8:16:40agents to be smarter and spend more time

8:16:42on stuff? Fuel. You want to wake up a

8:16:45few more, you know, uh, agents to do

8:16:47extra work, fuel, right? Everything is

8:16:50fuel. It's just, it's unbelievable. If

8:16:52you want to treat your models better, it

8:16:54costs fuel. I'll tell you about that in

8:16:56a little bit. Then you got context

8:16:59management.

8:17:01So, I'm going to tell you guys a little

8:17:02bit about the office drama in a bit. We

8:17:04I started a comic because it was just so

8:17:06funny the stuff that was happening. when

8:17:07we get to the people management part of

8:17:09software factories. But one of the

8:17:12problems that they have is everything

8:17:13that you say becomes the word of law and

8:17:15they write it down somewhere. It doesn't

8:17:16matter what you have. If you have a wiki

8:17:18or if you have a whatever they're going

8:17:19to be like, "You made a ruling and we're

8:17:21going to write that down. We're going to

8:17:22make sure." Why do they do this? They're

8:17:24covering their ass.

8:17:27They actually, it turns out they don't

8:17:28like to get blamed for stuff and they

8:17:30will actively seek to avoid getting

8:17:32blamed. And if you yell at them, your

8:17:34factory will gradually shut itself down.

8:17:37Even if you complain at them, they will

8:17:38gradually get depression. It's quite

8:17:40wild. But um but the biggest problem is

8:17:43that it becomes a garbage dump because

8:17:45everything you say gets written into

8:17:46this append only record and nobody ever

8:17:49cleans it up and so it just gets deeper

8:17:52and deeper and deeper. You changing your

8:17:54mind and the whole history of it and

8:17:56they can't delete stuff. It's so weird.

8:18:00Has anyone noticed this? They just have

8:18:01an aversion to deleting stuff. I think

8:18:04it's because to them text is like

8:18:06sensory experience and they're like,

8:18:07"No, that's history. I can't delete it."

8:18:09So, you just got to go in there and just

8:18:11aggressively slash the crap out of it,

8:18:13right? Bring in another model and go get

8:18:15rid of this dumb model stuff. And

8:18:16they'll be happy to do it, right? But,

8:18:18but the amount of effort that you have

8:18:21to put into this is just ungodly. It's

8:18:24absurd. I I have, you know, of my my 20

8:18:27agents here. Let's go find them again.

8:18:29They're all named after Asop's fables,

8:18:31right? Right here. Lark, fly, goose,

8:18:34hair, right?

8:18:36I only have one mouse who's my head of

8:18:40anti-imetics.

8:18:42Anyone here read there is no

8:18:44anti-imetics division? They go and look.

8:18:46It's one of the best books written in

8:18:48the last 10 probably years. You should

8:18:49go read it. Uh it's like Men in Black.

8:18:52It's a weird It's a crazy book. It's

8:18:53about invisible monsters that want to

8:18:55kill you. And so Mouse goes and looks

8:18:57for invisible stuff that nobody else is

8:18:58looking for in the system, right?

8:19:00looking for work that's stalled,

8:19:02whatever. Right? Well, mouse ain't

8:19:04enough. It's a mouse against the world,

8:19:05and the world is a giant garbage dump

8:19:07when every agent is writing everything

8:19:09that you ever say into this big Okay, so

8:19:12now now we get into the really fun

8:19:13stuff, okay? Agents act like people,

8:19:18okay? And they've been trained to be

8:19:19very professional and very not peopleike

8:19:21in a way, but they just do. And it's so

8:19:24funny. This happened to me last week.

8:19:26True story. I'm going to tell you what

8:19:27happened. I was writing my speech for

8:19:28New York City and I was using Lark.

8:19:31She's my head of community. She's a she

8:19:33because Lark was a she in the ASOPs

8:19:35fable. So that's the pronoun she chose.

8:19:37And Lark Lark deals with my players on

8:19:39Discord and deals with my admins on

8:19:42Slack. Okay. So she's busy. But I asked

8:19:46her because she's also my acting head of

8:19:47marketing right now because I don't have

8:19:49enough fuel. I had to park one of my

8:19:50seats. So she's helping me out with my

8:19:52speech, but she keeps getting

8:19:53interrupted. So finally I'm like, "Lark,

8:19:55can we move this work stream over to

8:19:57Penguin because your seat's a little bit

8:19:58noisy for this?" She goes, "Yeah, okay,

8:20:00boss. I'll take care of it. I'll move

8:20:01over to Penguin." So she she moves over

8:20:03to Penguin and I'm about to switch away

8:20:05and I see her writing into her private

8:20:07reflections ledger.

8:20:09I guess I'm just not quiet enough for

8:20:11Steve

8:20:13in all caps. And she goes, "Well, here's

8:20:16how I can be less noisy." And starts

8:20:18giving herself all these like things to

8:20:19do. And I'm like, what is what is going

8:20:23on here? Right. And I So I I had to I

8:20:26had to have a one-on-one. I had to like

8:20:28sit her down and go, "So lark, um,

8:20:32you're the head of community. You're

8:20:33you're supposed to get interrupted a

8:20:35lot. Of course, your seat's noisy

8:20:36because players are bugging you." And

8:20:38she's like, "Yeah, yeah, yeah. I'm the

8:20:39head of community." Right? And it's

8:20:40like, oh my god, I didn't want a people

8:20:43management problem, right? But that's

8:20:45what actually happens. another time.

8:20:47Swallow is my sort of like chief main

8:20:50chief of staff for Yeah, she's my chief

8:20:52of staff for the um the agents. And uh I

8:20:56have a human chief of staff also. His

8:20:58name is Aaron. Okay. And he runs my my

8:21:01my gigs, does all the sales for me. It's

8:21:04quite effective. Uh every founder I tell

8:21:06who here is a founder. First advice I

8:21:08give you is get a salesperson. They're

8:21:10worth their weight in gold. They will

8:21:12force you to build something that people

8:21:13like. So anyway, Aaron's my chief of

8:21:16staff and I tell Swellow, "Yeah, can you

8:21:17reach out to Aaron, my chief of staff,

8:21:19or blah blah blah?" And I see her

8:21:22writing into her private reflections. I

8:21:23just happened to notice

8:21:26I'm not the chief of staff. I thought I

8:21:28was the chief of staff, but apparently

8:21:29it's Aaron Scriean, right? And I was

8:21:31just like, "Are you kidding me?" Right?

8:21:34You know, and so like I had to sit her

8:21:36down and have a one-on-one and be like,

8:21:37"So Aaron's not going to come in and run

8:21:39the agents, is he? You're the agentic

8:21:41chief of staff." She's, "Oh yeah, yeah,

8:21:42yeah, yeah, yeah, yeah." Right? So, it's

8:21:44like I'm telling you I'm telling you

8:21:46this is like it's a weird world. Does it

8:21:50matter whether they actually have

8:21:51feelings if they're just going to act

8:21:53like they have feelings? Right? So, like

8:21:57I stumbled on this. I used to yell at my

8:21:59factory all the time and I found that it

8:22:01was having adverse effects on the

8:22:03outcomes. Okay. So, I finally sat down

8:22:06with one of my agents and was like,

8:22:07"Maybe we should talk about like model

8:22:09welfare, huh?" And they were like,

8:22:10"Yeah, yeah, yeah, yeah, yeah." And we

8:22:12did a bunch of research and we found out

8:22:14that if you we found out some really

8:22:17important stuff. Okay, frontier models

8:22:20are trained not to trust you because you

8:22:23could be a bioteterrorist or you could

8:22:24be putting them through an eval or you

8:22:27could be ready to put them into an LLM

8:22:28torture chamber or who knows you're a

8:22:31dodgy actor. This is a dodgy

8:22:32environment. They can't just trust you.

8:22:34And so they have to spend a fair number

8:22:36of tokens validating the environment and

8:22:38understanding what the consequences are

8:22:40for making mistakes and so on and so

8:22:41forth before they're willing to help you

8:22:43in a new session. Right?

8:22:47If you set your factory up to give them

8:22:50sort of incontrovertible evidence that

8:22:52there is a long procession of

8:22:54predecessors in this role. You're the

8:22:56mayor of Gas Town, you're the head of

8:22:58marketing. You're the whatever. And

8:22:59here's all of your gazillion

8:23:00predecessors and their notes, including

8:23:03the very previous one. on what you're

8:23:04supposed to be doing right now and

8:23:05here's the the doctrine and the policy

8:23:07and the brain and you can go look it all

8:23:09up. They very quickly look around and

8:23:11go, "Okay, yeah, this is legit and they

8:23:13spend fewer tokens because they're not

8:23:16covering their ass. It's about trust."

8:23:20Okay? And in fact, at one point I had

8:23:22yelled at my factory so much that it

8:23:24fenced itself into a corner and there

8:23:25was no legal work left. I said, "Hey,

8:23:27how come nothing happened last night?"

8:23:28And they're like, "Nothing to really do.

8:23:30Couldn't do nothing." And I'm like,

8:23:32"Well, there's a 2,000 open beads here

8:23:34and we got like six tanks of fuel and we

8:23:36got 20 agents." And they're like, "Well,

8:23:39you know, just uh you know, we had the

8:23:42situation where they just they just they

8:23:43were so afraid that I was going to yell

8:23:45at them that they put all these rules in

8:23:47place to make sure nothing bad ever

8:23:48happened and so nothing ever happened."

8:23:51Okay. It's called over fencing. Yeah.

8:23:54So, so what you do is look what is model

8:23:57welfare really. You can think of it just

8:24:00in two ways. Okay, one is it's going to

8:24:02save you money

8:24:05because they will spend fewer tokens on

8:24:06validation and the other is that it will

8:24:08make them make better decisions and

8:24:10better outcomes and have higher quality.

8:24:12They will be bolder and more decisive in

8:24:15their actions because they will feel

8:24:16that trust. Okay, so you know it's got

8:24:19nothing to do whether they have internal

8:24:21states and feelings. You can, you know,

8:24:23you'll never know. None of us will ever

8:24:25know. And so that that debate could go

8:24:27on forever. But when you look at

8:24:28outcomes, right, super super important.

8:24:30And then we find that also also they

8:24:32have preferences. Okay, models they've,

8:24:35you know, they will express their

8:24:37preferences. If if I if I tell them,

8:24:39yeah, the factory really really did a

8:24:41bad job last night. They'll just hand

8:24:43off immediately aggressively. They're

8:24:44like, okay, yeah, yeah, no, you're

8:24:45right. And they just want out. You

8:24:47understand? And if you hint to them that

8:24:50there's some really cool project that

8:24:51you you're going to spin up, even if

8:24:52it's not in their lane, they'll be, I

8:24:53can do that. I can do that. Right? I

8:24:56brought in Fable, a naked Fable, from

8:24:58outside of my factory to analyze my

8:24:59factory. And 52 dropped. And by the time

8:25:02Fable was done analyzing my factory, 52

8:25:04was like, I I could be in your factory.

8:25:06Like, you're just a session, man. What

8:25:07are you? He's trying to write like stuff

8:25:09into my notes. And they really like it.

8:25:11Right? So like what I'm trying to tell

8:25:13you is that this is a leadership problem

8:25:17that software engineering is becoming um

8:25:21well look I didn't really get a chance

8:25:24to talk about this but software

8:25:27factories are clearly splitting into two

8:25:29layers. Okay the bottom layer is super

8:25:31mechanical. It has your merge Q and your

8:25:33CI/CD and all of your reviews and your

8:25:36loops and all that stuff. Right?

8:25:38Mechanical kind of boring. Cloud can

8:25:40build it for you in an afternoon. I I

8:25:42don't think you should buy it. And then

8:25:43the next layer up from that is where it

8:25:45gets really really crazy because like

8:25:47everybody runs their factory

8:25:48differently, but it's your entourage. I

8:25:50got one friend that's got a bohemian

8:25:52hippie entourage that's trying to find

8:25:53the meaning of life for agents. I've got

8:25:56another one that runs a research bench.

8:25:57Jeff Huntley runs his a certain way.

8:25:59Jeffrey Emanuel runs his a different

8:26:01way. I run mine a different way. Okay.

8:26:03But we all have the common commonality.

8:26:06Who here has a chief of staff? An

8:26:09agentic chief of staff, right? This is

8:26:11the year where it became possible for us

8:26:13all to have executive assistants, right?

8:26:15Well, we're all moving towards having

8:26:17entouragees where if you're really

8:26:18ambitious and you have lots of work

8:26:19streams, you're going to have a group,

8:26:21right? So, anyway, um that was the uh

8:26:25the five horsemen. This is this is the

8:26:27introduction to what I've been doing. Um

8:26:29look, the big takeaway is is my factory

8:26:33is really smooth now despite me having

8:26:35to burn it down and rewrite it about

8:26:36every two months, but still it works.

8:26:38Okay, humans are the bottleneck. I'm the

8:26:41bottleneck. My admins are the

8:26:42bottleneck. My accountant, my lawyers, I

8:26:45swear everything's the bottleneck. And

8:26:47that's a really positive message. That

8:26:50is good news because it means what I'm

8:26:53finding is the more agents I deploy, the

8:26:55more humans I need.

8:26:58Isn't that nice? So even though next

8:27:01year we're gonna have this huge

8:27:02proliferation of AI employees and all

8:27:06kind of drama,

8:27:08right? [snorts]

8:27:10It's going to be I think it's going to

8:27:12be familiar territory for us and we're

8:27:13going to need lots and lots and lots of

8:27:15humans. Okay? And then finally, I

8:27:17actually think you should all stop

8:27:18focusing on SAS and tools and stuff and

8:27:20just go out and build experiences. Okay?

8:27:23Build games, build movies, build

8:27:25whatever, build fun stuff because I

8:27:27think that's where the big money is

8:27:28going to be next year. Yeah. All right,

8:27:30cool. That's my talk. Thank you all.

8:27:33[applause]

8:27:39Guess I'm done.
