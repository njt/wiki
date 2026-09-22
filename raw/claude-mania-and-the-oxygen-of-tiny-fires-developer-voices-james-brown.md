---
url: https://gist.github.com/njt/d4b2116bae2c74f3ffc458a130c4a4fc
date_fetched: 2026-08-14
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Productivity mismatch: AI turbocharges individual side projects, but enterprise gains lag because organizational bottlenecks (approval processes, technical debt, risk aversion) remain. AI amplifies existing messes—if your company is already on fire, AI pours on oxygen.

Claude Mania and burnout: The addictive rush of 24/7 AI-assisted coding leads to unsustainable cognitive load. The ability to leave agents running overnight creates anxiety about “missing out” on productivity, and burnout is already rising among developers.

Innovation cost collapse: The cost of trying ideas has plummeted. Companies should relax innovation gatekeeping, run many cheap experiments, and get better at killing bad ideas fast. The bottleneck is no longer coding speed but clarity of business problems and proximity between tech and the business.

Team composition shifts: Small, multi-skilled teams (2–7 people) will persist due to communication limits. The balance tilts toward “builders” (focused on solving user problems) over “crafters” (focused on language/machine perfection). Builders thrive when paired directly with domain experts.

Junior talent pipeline at risk: Hiring freezes and vanishing graduate programs threaten the next generation. The old instructional advice (“learn Java, learn OOP”) is obsolete; mentoring must become reflective and contextual, but we don’t yet know what foundational skills will matter.

Emerging paradigms: The “high throughput evolutionary harness” defines what “good” means via metrics, then lets AI agents autonomously generate and test features to improve those metrics. A “dark factory” repo can automatically process backlog items. These shift the developer’s role from writing code to designing measurement and feedback loops.

Non-engineers building software: PMs, BAs, and others are now spinning up prototypes, but they often underestimate the complexity of productionizing, leading to expensive token bills and fragile code. The skill of refactoring “vibe-coded” projects into production-ready systems will become critical.

Management expectations danger: Flashy AI demos can inflate leadership expectations, pressuring teams to sustain unsustainable pace and risking widespread burnout.

Pithy and provocative quotes

On the addictive pull of AI: “I realized this anxiety was like, my God, you know, you’re literally, you’ve got—think of all the people who have got a session running at home that’s starting up a new unicorn startup or solving a problem or a disease. And I’m just, you know, what a loser. I’m just a single-threaded man kind of hunched over, plodding to the shops.”

On AI amplifying organizational dysfunction: “AI is almost like giving oxygen to a room of tiny fires. … AI has come along, accelerated everything and flooded the room with oxygen and, you know, now it’s almost like there’s a catastrophic backdraft of that transformation you’ve been putting off for years.”

On the shift from crafting to building: “The builders are having a really good time right now because they just need to be hooked up to somebody who’s got a problem, someone to save. … The builder comes along and now they can have that dopamine hit of solving their problem much quicker.”

On the danger of inflated expectations: “You’re going to have a lot of people who get the Claude Mania … and they are going to produce things that look incredible. And yeah, I think a lot of people need a reminder that to operate things and iterate on it and turn it into something that’s sustainable is still a skill and it takes practice and process and rigor.”

On the new abstraction: “One of the things AI has brought upon us is probably the biggest abstraction that we never asked for since maybe compilers. … We are not going to be coding or coding is going to become an incredibly increasingly rarer and niche activity.”

Tools, practices, and methodologies

Claire (Clairvoyance): A Claude plugin that gives multiple agents working in the same repo “proximal awareness” to avoid merge conflicts. Agents periodically push small summaries of their current file locations and tasks to a shadow (orphan) branch in Git; if two agents get too close, a proximity alert triggers direct communication. Uses progressive disclosure (like skills) to keep context small. No server required—the remote repo acts as the communication backend.

High throughput evolutionary harness: Define a measurement harness that quantifies what “good” means for your system (e.g., frames per second, trade reaction speed). Then let AI agents autonomously generate features, test them against the harness, and iterate. The loop is: measure → generate → test → keep improvements. The only limit is feedback loop speed.

Dark factory repo: A repository pre-loaded with agents, skills, and guidance that can automatically process a backlog of feature requests, building and merging them without human intervention. Some practitioners even feed competitor feature analysis back into the backlog.

Intentional prototyping (the “wire terp” technique): Before spraying tokens, take a short deliberate pause to think about what exactly you’re building. Being a little more accurate and pointed makes you faster at delivering value over time, even though raw speed is seductive.

Multi-variant UI testing with AI: Instead of debating UI designs, generate multiple clickable prototypes with AI and test them with users. Throw away the ones that don’t work—the cost is now trivial.

Progressive disclosure for agent context: Borrowing from the success of Claude’s skills, share only minimal context (file globs, task summaries) by default, and expand only when agents need to coordinate.

Unanswered questions and omissions

How to balance speed and regulatory caution: The talk acknowledges that financially regulated companies must move slower, but offers no concrete framework for deciding when to accelerate vs. hold back. The risk of being “two steps behind” is mentioned but not operationalized.

Burnout prevention at scale: Personal tactics (a hammock, forced disconnection) are mentioned, but no organizational strategies for preventing AI-fueled burnout are proposed. How do you set sustainable expectations when the tooling enables 24/7 output?

What should junior developers learn?: The old curriculum is dead, but no alternative is offered. The conversation stops at “reflective mentoring” without specifying what foundational concepts (systems thinking? prompt engineering? measurement design?) will replace coding fundamentals.

Security of agent-to-agent communication: Claire’s shadow branch communication needs encryption/key exchange to prevent unauthorized access—acknowledged as unsolved and deferred.

Effectiveness of Claire unproven: The project currently consists only of a measurement harness and a README; no data exists on whether proximal awareness actually reduces merge conflicts or token waste.

Economic and environmental costs: Weekend agent runs burning thousands of dollars in tokens are mentioned as a caution, but the talk doesn’t explore the sustainability of such practices or the carbon footprint of high-throughput evolutionary loops.

Ethics of autonomous feature copying: The “dark factory” that scans competitors and auto-implements features is presented neutrally. No discussion of intellectual property, market ethics, or the race-to-the-bottom dynamics this could create.

What happens to the “crafters”?: The talk implies crafting will become a niche, but doesn’t address whether deep systems knowledge will still be needed for debugging, security, or performance-critical work when AI handles most code generation.

How to refactor “vibe-coded” projects: The need is identified, but no methodologies or tools for safely turning AI-generated prototypes into production systems are discussed.

Speaker A: How much is AI going to change programming? And I ask because I see a discrepancy in productivity. Individually, I think it's making massive changes. I've got friends of mine who are career long programmers and they're no longer writing code, but they're shipping more code, working code than ever before. Personally, I've ticked off so many side projects and plugins and extensions and things that I've always wanted to build but never had the time to. It is a great time to be a builder or a tinkerer in this market, but at work I don't think it's quite the same story. There's more noise in the enterprise about AI, but I haven't seen the same explosion of productivity. Why not? Maybe it's that AI can be trusted for side projects, but not for production code. Maybe coding faster doesn't actually help if the road to production is filled with lots of other bottlenecks. Maybe some companies aren't really doing it right, they're just pushing their staff to burn more tokens and not thinking beyond that. But somewhere there is a mismatch between the results we're seeing at home and the results we're seeing at work. A few weeks back I was asked to chair a panel at the conference XT26 and the topic was how far can companies accelerate with AI companies specifically, the answers from the four panelists were very interesting, very mixed. But if there was a consensus, it was that the current state of your company is probably going to predict your success with AI. If things are already a mess, AI has a real risk of making it worse. One of the other things I took from hosting that panel was that I can't get four interesting people in a room and only give them an hour between them. I need more. So I'm trying to get all four of them come in and join me on this podcast. My guest this week was the first to say yes. I'm joined this week by James Brown, who is an engineering lead at Schroeder's Asset Management and I think that company specifically puts him a good vantage point. Asset management companies, hedge funds, places like that, they often have all the regulation and organizational headaches that banks get, but they're mixed with those. Move quickly, be the first to market opportunities that you see at startups. It is a good ground zero case study for the AI explosion. In talking with James, we talk about how Claude Mania has grabbed him and put him at risk of burnout, how it's got him building a new system for context management for Agents. That reminds me quite a lot of the way human beings learn about a code base. We talk about how developers will have to change mindset going forwards and how teams will need to change their organization. And we talk about the near future, what's going to happen a few years from now if we aren't training junior developers anymore. And what are the risks to our careers if our managers are bluffing their way through the revolution. Sometimes opportunity knocks and sometimes it brings a battering ram. Will your career and your company be able to cope? I'm your host, Chris Jenkins. This is Developer Voices and today's voice is James Brown. Joining me today is James Brown of Schroeders. James, how you doing?

Speaker B: I'm very well, thank you for having me. How are you?

Speaker A: I'm good. I think we're both surviving the heat in the UK today. We're hitting record temperatures, right?

Speaker B: Yeah. It's hot again. Yeah. I mean, I quite like it, to be honest.

Speaker A: Yeah.

Speaker B: But yes, unusually hot again.

Speaker A: I'm a child made for the winter. I much prefer it. But speaking of things that are hotting up, how's that for a link AI in the world, which is the hot topic of the day and something you're experiencing both at the coalface and the corporate level. So I thought we'd get you in to chat about it. Why don't we start with your experience with things like Claude? Because I think you had a similar one to me for in that last year. You're wondering if this was all hype, but things changed.

Speaker B: Yes. Yeah. I mean, I think this is felt by a lot of people that I know in my sort of circle and where I work and the previous company as I've worked is there was a long period of time where the, the hype seemed to be outstripping the reality. And for those kind of more sort of scientific, analytical people, we're kind of watching and reading the rhetoric and looking for the numbers and things weren't adding up. But there was a specific point in time for me where I realized that, yeah, this is, this is not just here for state to stay. This is a big deal. Everything is going to change. And for me personally, it was about the, the back end of 2025, sort of, you know, maybe autumn, winter, and Claude was kind of gaining some traction. I tried a whole bunch of AI tools and I thought, you know, I've got hundreds of unfinished personal projects as a lot of, a lot of us do, you know, libraries. I work been working on this, this artificial life simulator for 20 years. And, and I thought, okay, Claude, show me what you can do. And you know, I'd been learning kind of, you know, it wasn't just kind of. I was more than one shotting at the time. You know, I was exploring proper sustainable AI engineering practices. And I thought, you know, I picked it up at home and I thought, okay, let's throw it at my biggest project, the toughest one. The one that is always like the measure of what I can and can't do.

Speaker A: Right.

Speaker B: And what happened was nothing short of like what I call Claude mania. I just not only did I smash all of my previous goals with my personal project in days, but I couldn't put it down. It was almost like, you know, you know, it was addictive.

Speaker A: Yeah.

Speaker B: I kept throwing it one more feature, one more thing to do, one more improvement, one more millisecond of performance. And it went on for a few months. And I would have multiple projects on the go, multiple terminals, agents running, switching backwards and forwards. For the first time in a long time. I went back to that stage that I did when I was very young in my career where, you know, I'd be up till three in the morning coding still.

Speaker A: Yeah, yeah, yeah.

Speaker B: And, you know, this went on and there was a point in time to come in towards Christmas and I thought, you know, like, I think we were out of milk and eggs. And I thought, go out, get out of your, get out of your little cave, go outside and get some milk and eggs.

Speaker A: Right, right.

Speaker B: So I did. I kind of crawled out of my, my little basement flat. There's some sun burning me because I hadn't seen daylight for months.

Speaker A: You know, like a Morlock in the HD world story.

Speaker B: Exactly, exactly. Like a warlock. That's exactly how I felt. Right. You know, it's kind of hunched over and the sun was bearing down and I'm walking to the shop and all of a sudden I felt this wave of anxiety and I was like, what is that? And I realized I hadn't left like an agent or a session running at home doing something. And I realized this anxiety was like, my God, you know, you're literally, you've got, or think of all the people who have got a session running at home that's starting up a new, you know, unicorn startup or solving a problem or a disease. And I'm just, you know, what a loser. I'm just, I'm just a single threaded man kind of hunched over, plodding to the shops. And I actually still get this today. So obviously that was the beginning of Using AI all day, every day. But if I leave work and I haven't kicked off a bunch of agents to do stuff while I'm away, I feel like I've kind of, you know, made a mistake or missed out.

Speaker A: Yeah, it is. I mean, I can imagine saying that in like 2024 and it's sounding ridiculous, but I really, I really know what you mean. Like, I will often leave things to cook overnight so that I've got something done waking to wait when I wake up in the morning. Right.

Speaker B: Yeah, it just, it just feels like you're missing out on. On some level of productivity. I think we've all been given this ability to be productive 24 hours a

Speaker A: day and that, that is a blessing and a curse. Is that translating into work where you're expected to be productive 24 hours a day?

Speaker B: No, I mean, I think they're still trying to work out, you know, what's, what's safe and sustainable as well. Because that kind of Claude mania was, wasn't really sustainable. It wasn't a healthy thing to do. And I think you'll have experienced this. And I have a lot of friends who are more burnt out than ever, especially cognitively. Yes. So I think there's going to be unsustainable, you know, practices before we work out what a sustainable and healthy 247 agencic world really looks like.

Speaker A: I have to ask you, of all those old projects you picked up, how many of them are now like, complete, still useful, you're still working on, and how many were like part of the excitement but died?

Speaker B: Yeah. So did it change the fact that I can't finish a project? Is that what you're. Yeah.

Speaker A: Did it actually produce external results or did you just get busy and noodling?

Speaker B: Well, I mean, at work I finished, you know, significant things. Right. A home project, it's always best effort, but I feel like I could finish them. I feel like I could. Yeah.

Speaker A: You reach the point sometimes where, especially when you've got kids, the amount of spare time diminishes to almost nothing and you don't have even a feeling, a smidgen of power.

Speaker B: Yeah. You don't control how much time is available, nor would I want to edge out the unpredictable sort of demand for doing that kind of family stuff. So. So I really feel like I could finish some home projects now. But maybe that's just reflective of a lot of the, you know, the productivity in AI is everyone feels more productive. But when you actually start reaching for, for any kind of evidence, it is still Quite tricky.

Speaker A: Yeah. I sort of feel like personally the evidence of being more productive for getting more things done is undeniable. And probably the things I've released, there are more of those. And yet, and yet what is it I'm grasping at that I don't quite feel. Firstly I don't quite feel that there's been a revolution in the valuable things to the world I've delivered. Mostly it's me noodling and I'm not convinced it's entirely translated to the corporate world.

Speaker B: No, I mean I think there's still a lot more to actually delivering software than finishing a project or the code. And you know, I know this through professional terms for a lot of the things I'm building that you build it because you want someone to use it. Right. They either, you know, not necessarily buy it, but if it's an open source project, you want people to fork it, download it, use it, feedback, contribute. Right. And I've been involved in a number of both professionally and private sort of open source projects and they are all, they were, all the complexity was about getting engagement. So talking to people, sharing people, getting them excited about it. So unless you're using, and I'm sure you can use AI to help you with that, but I think that's still there. Like I think if I could finish my, you know, I've got an open source thing I'm working on now which is for trying to give like proximal awareness or, or sort of agent proximity alerts to multi agent things. Unless I know, unless I actually go out and get people interested in it and join in and try it and, and that it's, it's still always just going to be a GitHub repo with, with one user myself. Yeah. Even if I would declare it, you know, complete.

Speaker A: I'm going to pull on that thread partly because we've got a perfect platform to let people know about this thing that you think is interesting. But what's a proximal awareness system? What are you trying to build on there?

Speaker B: Yes. So it's. So it's called Claire.

Speaker A: Right.

Speaker B: And that might be because it's called. It's short for clairvoyance. Now the. Let's take a hypothesis and I don't necessarily believe this hypothesis is true. It might be validated. Let's take a hypothesis that we are now going to have a world where repositories, single repositories, especially for companies that have large mono repos, maybe like Google, maybe they'll pick this up and it will change the way they work. Will have at any one time, many, many agents working more than we ever used to within the same refi. Now at the moment, obviously what happens typically is somebody orchestrates the work and you try to divide up the work to the agents in a way that they're not going to cause merge conflicts. So you wouldn't set 10 feature agents off and an agent that's going to refactor the auth flow and an agent that's going to upgrade all of your out of date libraries, they're all going to come up with a horrendous merge conflict and then another agent has got to do something.

Speaker A: Yeah, that makes sense.

Speaker B: So the, the idea is what if you could give all of your agents some kind of like skills are one of the big sort of successful things that were added to the ecosystem and one of the reasons for that is because of their progressive disclosure. So they, they share a very little bit about of what they can do to your context. And when you sort of trigger that you need to use that skill, then it pushes the detail into your context. So what if we took that approach with agents doing live work? So the idea was that all of the agents would store like a little front matter about what files they're in, like the globs.

Speaker A: Right? Yeah.

Speaker B: And roughly what they're doing and they would all be aware of that. And the way I've done that, because I wouldn't want a server, is the plugin has a shadow branch in Git. Essentially the GitHub or remote repo is the server. As they're working, they're going to push these little front matters of the proximity, where they are and what they're doing.

Speaker A: Right? Yeah.

Speaker B: And they're all aware of it because the Clare plugin is going to constantly be looking at what the other agent is in this file over here and it's roughly working on auth this one's over here and it's working on this feature. What it does is the if it has a proximity alert, so if any two of them get close, they a proximity alert is triggered and the two agents get in touch through the same shadow branch and share what they're up to.

Speaker A: Right, right. Yeah. Because they're all working on separate branches, they aren't going to immediately step on each other's toes, but in the future they certainly will.

Speaker B: Exactly.

Speaker A: Yeah.

Speaker B: Yeah, exactly.

Speaker A: That's a really smart idea.

Speaker B: And then the idea is so all of the, you know, I know this from experience, like building it is one thing, but Building the, the measurement harness to prove it works is where all the work is and so that's, that's all I've built so far. It's because it's very complicated thing to measure is to set up the environment in which I think this provides value. So so far it's a readme and a measurement harness that tries to create this sort of the multi agent doing multiple things and measuring, you know, the tokens used, the merge conflicts, the time resolution time. So I just have a very sophisticated harness and an idea and we'll see. You know, I guess I will finish it off and share it maybe as a comment in this, in this video and I'm sure somebody will come will tell me oh we've tried that and it doesn't work but it's fine.

Speaker A: I suspect you'll find that a few people are trying it at the moment and they all have slightly different designs to yours and who will be right. Right.

Speaker B: Yeah there is a number of kind of agent to agent sort of context sharing things out there and you know I, I think my sort of take on it was using the, essentially using the progressive disclosure that was successful in skills and using the shadow branches or you know, orphan branches as the back end means that both people can pick it up without any infrastructure. And maybe the sort of the progressive disclosure of the proximity system is, is something that's worked. I mean it's the testing harness is the thing that flushes this out.

Speaker A: Yeah, yeah. You're going to burn some serious tokens actually running tests on this. But yeah, the thing this makes me think of is the reason you would need something like that is traditionally the way we've discovered this that I'm working on the AUTH system and some other guy on the team is working on the user account page. Right. And they're going to clash and the way it works usually is I overhear them talking about it.

Speaker B: It's kind of the same thing, right?

Speaker A: Yeah yeah. So that makes me think there is an exact analog between we're finding ways for agents to work that mirror the way humans have always sorted this stuff out. Right.

Speaker B: Yeah maybe that's, maybe that's a, a good thing to do or maybe an anti pattern to apply but we're going to find out I guess. But yeah, I mean it's exactly like you said. It's not just that is the person touching your system may not have initially been tasked to touch your system but they had the hole in the bucket problem. If they went to fix a bug and Then they found a bug within the bug and the next thing you know they're refactoring the entire AUTH system. So it's not like you can do it from the backlog. Right. The backlog will probably not say they are upgrading React or they are in the AUTH system. And it's almost like the merge point, depending on how the company is doing merges and branches, feels also too late. So maybe it's adding something in the middle.

Speaker A: Yeah, yeah. I've got to ask, explain this shadow branch mechanism to me. Is it like in the background constantly committing and pushing work in progress or something?

Speaker B: Yep. So from a, from a sort of git technicality point of view, I'm not going to be your expert here, but I heard about its use in a few other libraries. So essentially you could get the communication through remote git, so no server. And I thought I'd take the same approach. I believe it uses like an orphan branch. Somebody in the comments in the video is going to have to give us the details. Okay. But it means it's not. It's a kind of an ephemeral branch on the git remote and you can use it for, you know, the back end communication essentially.

Speaker A: Okay, that makes sense. Yeah. And what you're automating pushing to it or automatically does that.

Speaker B: Yeah. So they. The plugin only works for Claude at the moment and it uses hooks so it will kind of, it maintains like a, a summary, a small summary of what it's up to and it will periodically sort of push that up to the shadow branch and the other plugins periodically kind of fetch and then if there's something new they'll pull it down. But it's only very small amounts of information like to give like a position, a positional awareness really. There's not a huge amount of extra traffic. If they need to get in touch then that would be obviously a lot more sort of track across your remote repository.

Speaker A: Yeah.

Speaker B: Also the plan is there. Right. Is probably because of where I work, but you would need to do kind of like a secure communication, like they need to share a key with each other so that anything on the branch is not necessarily revealed to anyone who just has access to the re. That's the other complexity. Probably only something I have to solve because of where I work.

Speaker A: Yeah, yeah. Working at a bank that you get these kind of enterprise grade problems, right?

Speaker B: Yeah, I mean I think they're good things to have. Right. Because I mean something like that is a good thing to do anyway.

Speaker A: Yeah, yeah, yeah.

Speaker B: That's.

Speaker A: I wasn't expecting we'd talk about that if I didn't. Didn't know you were working on it. But I can definitely see that there's. Because it's like, how is AI changing our structures, but how is it actually not changing our structures? And we're going to have to react. We're gonna have to translate existing structures into the AI world.

Speaker B: Yeah. And it's throwing up like so many cool new problems to solve, which is like what we, you know, a lot of people sort of, I think, misclassify devs or as programmers or coders, but really we're problem solvers and our, the computer is our favorite tool. That's how I've always felt like it.

Speaker A: Yeah, yeah, totally.

Speaker B: And I'm not on my own. A lot of friends of mine are building up. I was talking to a friend the other day and he's building something that is trying to give long term memory in a really interesting way. You know, using a graph structure and sort of embedding in a really smart way. Everyone's kind of seeing these new problems as an opportunity to have a play around. Yeah, yeah. Great excuse to learn more about all of this new tooling.

Speaker A: And it's also a great excuse to maybe come away from the keyboard a bit and learn a bit more about the problems with human structures that you get in an organization.

Speaker B: Right, yeah, absolutely. I mean, everybody. I think there's been a number of talks I've seen and a lot of writing is about how AI is amplifying the existing, you know, fire. I think I, I did a post the other day that a lot of people kind of, you know, found quite amusing is like if AI is almost like giving oxygen to a room of tiny fires. So a lot of these organizations where they've got all, loads of tiny fires, right. The organizational structure isn't quite right and they've got too much technical debt and their funding model is wrong and they're all tiny fires everywhere. And AI has come along, accelerated everything and flooded the room with oxygen and, you know, now it's almost like there's a catastrophic backdraft of, you know, that transformation you've been putting off for years. Maybe now, you know, you wish, you wish you had done it or you should do it immediately. But yeah, I think, I mean, certainly that, you know, certainly we are seeing the distance, the sort of the communication distance between technology and people in the business needs to shorten now. Right. They need to be in proximity, like, you know, sitting next to each other. Sort of as close as you can get to that. Because the, you know, the, this limiting factor isn't the speed that we can try ideas anymore. The limiting factor is actually the ideas coming through and the problems coming through in enough clarity so that we can throw ideas and products and technology at it. That's what I'm seeing.

Speaker A: I'm really interested in that because you're right, the cost of getting an idea to at least a prototype and cost in terms of money and in time has massively gone down. My experience of working for banking size organizations as you do is that ideas often die on the vine because you know, even if it gets picked up, that's probably an 18 month project that might fail.

Speaker B: Yeah, 100%. Yeah. And you know, I guess the trap is for people to not realize that the cost of innovation has just taken a huge dip. And a lot of processes have been set up essentially to really heavily controlled innovation that maybe need to relax, just let more ideas view and you need to get better at the end of the feedback loop where you kind of kill a lot of the ideas that aren't going to work and double down on the good ones a lot quicker.

Speaker A: So you're finding that in the real world that you've got new ideas springing up and the system trying to push against it.

Speaker B: Yeah, I think that's natural because these kind of organizational structures and the processes, the funding model is baked into it, everything, right, the culture and so it has ossified, I think it all. You know, before Schroders I did some consultancy for, for four years. So I saw a lot of big corporations, you know, works in government for you know, 18 months. So they, these things are ossified. Right. They are there and the immune system is not going to rewire itself easily.

Speaker A: Yeah. And often they're structures that sensibly existed. When a project would take 18 months, could fail and probably cost millions. It's not going to suddenly readapt to the new constraints.

Speaker B: No, no, of course. I mean like to see it's almost daily now. Somebody will be kind of debating over which feature they want to prioritize and what it should do and what it should look like. And you know, every now and then you can remind them, it's like, well if, you know, maybe do do all of the ideas that you're thinking of and have a look at them, you know, especially if you do them in a sort of a proof of concept, you can, you know, what should our UI look like? Well, let's try all of them. And see which one the users like and throw the other ones away. It's, it's not a big, it's not a big chunk of work anymore.

Speaker A: Yeah, that's now suddenly a sane strategy.

Speaker B: I think it was always the same strategy and I think the best companies were doing it.

Speaker A: Yeah, I, I think like five versions of a UI would take five teams, six weeks each. That was a crazy thing to do.

Speaker B: Yeah, but no, I mean, they wouldn't do it that way before, would they? They would basically do pull the wizard screen. Right. And use slideware and whatever. But now potentially you're right. You could literally have almost functioning, if not functioning, clickable versions of, of the variants.

Speaker A: Yeah, yeah, I think you can now. But so there is a tension there then. And it's probably one of the most important tensions in management right now. You're at home like speeding through all your side projects and your new ideas, probably being more productive than you've been in years as a home coder coming to the workplace. We know that should happen. The structures are fighting against it. But I, as someone answering to the shareholders up in management, have massive frustration that I'm not getting the gap between innovation I see in AI and innovation I see in my organization. And I would like to scream at someone or fire someone. What's your constructive suggestion instead for the way management should treat this stuff?

Speaker B: Yeah, well, I mean, I think we all know that everything has to change in financially regulated companies. There is an acceptance that it needs to be a little bit slower, I think because this thing comes with just, you know, huge amounts of extra unknown risk. We're not talking about known risk, we're talking about unknown risk that is flooding through this. But at the same point, you know, I mean, we've rolled out AI tooling to everybody, Schroders, and we have great adoption and it's going really well. And, and this kind of balance of how fast you, you kind of want to take and how much you want to slow down is, is both, you know, a lot of something that we didn't expect we were going to need to do not that long ago. Yeah, I mean, I mean, there's no, I think it makes sense to be one step behind for some companies, but it terrifies me to think of being two steps behind. I think it's, yeah, it's a super lineal scale. The gap between the companies that are going to learn how to do this well and the ones who are, who are going to be behind is going to get big really quickly.

Speaker A: Yeah, yeah. And Finding that midpoint. Because when you're a bank dealing with a lot of money, you do have to be more cautious. But you say it's going really well. I want you to give me some details on that. What counts as AI going really well in an investment bank?

Speaker B: Yeah, I mean, so adoption's been great. You know, I can't talk about specifics of what people are working on, but we measure token use, we measure adoption, we measure tool. We have people contributing significant parts of AI components like skills actively across the organization. We have products that have been finished end to end with AI and we're exploring how does this become a core part of a differentiator for clients as well. So, you know, it definitely.

Speaker A: Okay.

Speaker B: It definitely feels like, you know, when we look out and we kind of explore the rest of the industry, you know, we, we are, we do feel like we're, we're up there at the right, exactly the right pace, I'd say.

Speaker A: Are you finding that you'd get more projects delivered faster? Because that's the real crux of this.

Speaker B: So we probably don't really have enough data. I would, I would say that we're definitely seeing some projects come through really quickly.

Speaker A: I would have thought it's normally distributed, right?

Speaker B: Yeah, yeah. I mean there's certain parts of the estate that you do we wouldn't touch yet. So you know, there's a lot of being cautious and careful of where you point this out, but where we are pointing it at and where we've got it right with the right people and the right tools, the right approach bringing them closer to the business. We've seen phenomenal pace. We've seen things happen in weeks that would have been six months to a year and like you said, just probably not viable.

Speaker A: Yeah, yeah. There are plenty of projects died that would work if they weren't so incredibly complex and slow and expensive to produce. Right. The economics of producing software is completely changing. But you said like where we've got the right people and the right structures and stuff, what are those? Right. Like what's your initial hypothesis of what works for approaching this?

Speaker B: Yeah, I mean again it's very easy and completely reasonable at this point in time to say I don't know. But you know, best guess, I think it's kind of the same as what we knew from before. I think they're multi skilled teams that can both understand what the business needs. Whether that's understanding the users and their problems or understanding how the business make money combined with people who just love. Because I think there's. And I think, yeah, I've heard this before. I can't remember where I heard it. I'd like to attribute them, but there's kind of two kinds of different mindsets in engineering. Right. There's the crafters and the builders.

Speaker A: And explain that distinction to me.

Speaker B: So the crafters really loved the language. They loved to really understand the programming language and the way the machine worked. And they weren't really super interested in what they were producing is love the craft, right. The perfection of the craft. Right. And they're like, yeah, I've know some of these people and they're great. Right. They could always go deeper in a problem than I could. And then there are the builders who really, they're thinking about the thing they're going to hand over. Right. Somebody's got a problem that they think is unsolvable and because you know enough about how to program and software, you say, I can solve that for them and imagine their face when I hand it over. Yeah. And. And I've always been more of a builder than a crafter. I've been a bit of both. Right. And I think the builders are having a really good time right now because they, they just need to be hooked up to somebody who's got a problem, you know, someone to save. I've got a problem. How could we possibly solve this?

Speaker A: Yeah.

Speaker B: And, and the builder comes along and now they've, they can sit, they can have that sort of that dopamine hit of solving their problem much quicker. So you're pairing those with the people who understand the problem space or the user's needs or the business needs. And I think for a long period of time teams are going to be small, maybe just two or three people like that. That's our guess.

Speaker A: The number of people you need, I think is definitely going to go down. Do you think it's going to be about finding that right balance of personality types then to make a small, tightly performing team?

Speaker B: Yeah. And again, that wasn't really. Yeah, that was the same problem as before. And ultimately I don't think team sizes will change because of AI, because good team sizes were constrained by communication limitations. Right. Look at team topologies and small world networks, you know, like the amount of people that you can have kind of like a close relationship with. So the two pizza size team really comes out of a human limitation of how many people you can share a goal with. So I think they'll always, I think it'll always hover around that number. You know, one or two people is Always very risky because obviously you know, the, the, your lottery number is, is too low, you know, number of people who could win a lottery and all of a sudden you've got nobody operating that thing that's critical to your business. So I think feel like one or two is too low and I would say sort of seven and up is too high, but I think they'll hover around the same number.

Speaker A: Does that mean in your opinion that we're going to find that the level of hiring remains roughly the same over time?

Speaker B: Well, who knows? I mean it's surprising to me how little hiring there is going on right now, especially for younger people, which is obviously something we caught up on before. That's a worrying trend, right? And we've heard it from a lot of the major big software companies and we've seen the numbers and it's not just that there's layoffs, but you know, recruitment, especially for younger people feels to me in like it's a very worrying state.

Speaker A: What are you saying?

Speaker B: Well, I think, you know, we have a lot of people who, you know, for the first time in my career I have people who I've been out of work for months and months and they are the kind of person who would, would have been snapped up at a moment's notice if not already headhunters, you know, chasing their LinkedIn profile before. And that's obviously we're very worrying, but I'm also hearing across a lot of organizations that you know, sort of junior talent programs are just non existent anymore. Junior developers just don't seem to have, are looking for any new open roles. And you know, it's, it's, it's strange because I would have always thought of, you know, young talent, especially in this because I think one of the things, the way I see one of the things AI has brought upon us is probably the biggest abstraction that we never asked for since maybe compilers, I don't know, but it's this giant abstraction. We are not going to be coding or coding is going to become an incredibly increasingly rarer and niche activity. Our ability to share information backwards to the people who, the younger generation are going to figure this out and you know, they're going to probably run us all out of jobs at some point. And you know, I feel like there's, there's nothing set up for us to feed back what we've learned in a way to kind of help them do that without making too many mistakes along the way.

Speaker A: Here's the thing I think is really difficult about that because on the one hand, I think we are running the risk of ending up with a generation of people that won't learn the basic skills of coding like we did. On the other hand, I'm not even sure I know what the basic skills of coding are going to be in the next few years.

Speaker B: Yeah, yeah. I mean, it's easy to, you know, I think because it's all about what advice would you give to a young, you know, somebody who's maybe thinking about picking up software engineering as a career? What advice do you give to them right now? And I've always felt like the worst advice that was ever given to me was always, always came in instructional format. So, you know, remember when I first wanted to become a software engineer, I, you know, I didn't have a computer science background. I didn't do computer science at university. I did geology.

Speaker A: Okay.

Speaker B: You know, start, start at the, the silicon was the idea there.

Speaker A: Yeah. And the world will always have rocks.

Speaker B: Exactly. And when I kind of joined a company, I decided I liked programming. I could do it eight hours a day, which I couldn't do a lot of job, other jobs. And when I asked, you know, the person at the company I was interviewing for, who was, I think they were maybe their VP or, or a senior developer or something higher. And I asked them, you know, could I, could I do it? Could I become a software developer here? They kind of said, they laid out their, their exact trajectory of what they did as instructions of how I should do it. Right, right. So, yeah, and obviously there's, there's not just one way to get there. And I think they were mistakenly thinking, well, this is how I do it. You know, I'm obviously successful, so that's how you do it. So instruction step by step, much better advice that's always been given to me has been in reflective, honest, you know, what they learned, what they were, the context at the time, what they did, what happened, what they learned. And I think that's the only advice that translates. Now we can't give the instructional advice of learn object oriented programming, you know, learn Java. Right. Learn function program. That advice isn't going to work. It's almost like we need to kind of say, well, you know, I, I think I had a similar problem and, and this is how I solved it. And this is the context at the time. And you know, I think they're going to have to kind of work out how, how that works in the new world. But you know, we're, we're not done with our careers yet. You know, we're still, we'll still learn something from, about how AI does work over the next couple of years. So we'll be better mentors by then, I'm sure.

Speaker A: Yeah. And we'll get more used to it. I wonder if the advice we need to start giving people is, would, do you actually like building stuff?

Speaker B: Yeah, yeah, I think so. I think yes. Crafting is always going to have a niche. You know, some things need to be very finely crafted and I like crafting. You know, there's always that part of the system that needs to have the craft. But I think the overwhelming majority of demand is going to be coming to the builders, right? The people who want to deconstruct a problem and use computers through AI to solve that problem in a really compelling and interesting way. So, yeah, I mean, a lot of that is kind of taking them out of the low level craft of coding, which I guess is a shame because that has been so important to a lot of people for a long while. It's going to take them up to the whole loop. Right. You know, how do you just, how do you know the user wants it that way? Have you asked them? You know, maybe you can ask them and you can bake it into your agent's decision making process.

Speaker A: Yeah. I wonder if we're going to find that the craft changes not to how does it execute, but how well does it work for people? Because there's a lot, there's a lot of pleasure I found with working with AI when you've got this machine that will tirelessly change the software for you in being able to say, well, okay, now it works, but is it good? Does it feel good to use?

Speaker B: Well, I mean, so one, one thing, I'm just going to look up a term here because I think this is really relevant. Right. Okay. I don't know whether something that I've been using, I've used a couple of times and there's, there's some people I'm working with who've used it sort of covertly, used it without us talking and you know, this new approach to building software that I'm seeing, whereas rather than kind of, you know, rather than kind of defining what you want to build and then laying down the programmatic sequence of things that need to happen to build it. Right. The next level up, people are talking about like the dark factory where you create the, you know, create a repo with your agents and you create the skills and the guidance and then you give it feature, you give it link to your backlog and Everything on your backlog gets done, right?

Speaker A: Yeah.

Speaker B: But you're, you're still telling it what to do, right? The, the sort of level above that I see is this thing I'm calling like a high throughput evolutionary harness.

Speaker A: Okay.

Speaker B: If you can, if you can build the technology that, that measures what great means for your thing, whether that's, you know, for my game at home, it's all about, at the moment, it's about performance because I want to have, I want to like, it's, it's an artificial life simulator. So it's all about little creatures crawling around.

Speaker A: Right.

Speaker B: And it's all about how many, how many creatures I can get in there. And, and I've been trying to improve that for 20 years. I'm up to a million, a million creatures simultaneously at 90 frames a second in this.

Speaker A: Okay.

Speaker B: Yeah, so, so some people in algorithmic trading, it might be speed to react to information or trade, whatever, but if you can build the harness that can basically measure what good means and then you can give it like the dark factory ability to actually create sustainable, good features. If you can back, if you basically hook the input to the features to the output of the measure, then you have a sustainable loop of like, well, it's going to improve on its own. It's going to build itself.

Speaker A: Yeah.

Speaker B: And, and I think your only limitation then is how fast the, the feedback loop is. Right. So if you. Yeah, I've seen people build these sorts of dark factory repos that will have agents that scan the market for what they can, other features their competitors have added, and we'll feed it back in as a feature request to implement it all the way through and come back out on there. You know, they. Without anybody ever kind of knowing that it did that, right?

Speaker A: Yeah, yeah.

Speaker B: Our application just created a new feature today. Well, let's have a look. That's.

Speaker A: I mean, you've got to have a lot of trust in AI's ability to write code for that to work. But I can start to believe that we're going to develop that trust. It feels kind of sad that it means the value of a good idea is going to drop when it's so cheaply copyable.

Speaker B: Well, well, I think that was a good idea. I think they, I mean, I think, you know, if you think of coding as kind of, you know, building a shed or a house, then maybe. But if you think of it as tending a garden, then, you know, it's still a lot of kind of interesting engineering and thought and creativity gone into that. But it just Means that you're not necessarily, not necessarily going to know exactly what comes out the other end. But I've seen that's actually true of very top kind of SaaS companies. The way that they, you know, they don't define the features of what they're going to build for their products up front. They build one thing and they get that. They kind of create that hook, that loop with their users and then that loop follow through a very smart set of product people who kind of work out. Is that right for our, is that right for our product? Should we build that? That loop guides them to where they get. So I'm not sure that it's different. I think it's just, almost just there's more automation in that sort of ideal.

Speaker A: Yeah.

Speaker B: You can always throw your ideas in there along the way. Right. But it's subject to be tested and evaluated by the harness, you know, so you might be disappointed if the AI's idea, you know, comes out better than yours.

Speaker A: Yeah. Do you know what that makes me think of though? Do you remember that. Was it Malt book? So they had. Yes, they did like a Facebook for agents where they could all talk and post and it kind of sort of descended into this chatterbox of nonsense.

Speaker B: Yeah, it went south quickly. But I mean, is twitter or x.com any different? I'm not sure.

Speaker A: I mean certainly in some places it's absolutely no better. But it sort of again raises that question of what do we need to know to make the best out of these systems? Because I don't think we need to know how to do individual lines of code anymore. But there are skills we desperately need to not have this turn into a pile of slop.

Speaker B: Yeah, I mean right now the models are improving enough that I would not say anything I say right now is going to be a whole train three to six months. But right now you absolutely need to know good sort of best practice design patterns, what technologies and languages and frameworks are good for and not good for, you know, completely unattended. It still makes a lot of mistakes, a lot of bad choices. It doesn't seem to really like. It doesn't seem to know that its own sort of context limitations like messy code and duplicate code still trips it up the same as it used to trip us up. But there are, you know, it's just improving so quickly and the community skills and tool sets and plugins and the models and everything are so growing that I think there will come a point in time where it does most of the time make very good Programming decisions, I think. So right now it does need a very experienced handler still, I think.

Speaker A: Yeah, yeah, I would agree. I have to think about what the future is like when it really is just say what you want and it gets built. Does that, what then? Then the problem becomes how do you know what to build and who's good at that.

Speaker B: And I think, you know, we could all write a book, right. Like there used to be, there used to be kind of friction in the way of maybe writing and distributing a novel. Right. We could all do that right now online. That doesn't mean we're all suddenly international bestsellers. So I think, understanding like what are you going to do for who? Why would they engage with it? Why would they adopt it? Is there scale? I think all those things are still there. Yeah, yeah.

Speaker A: And I think all software developers are going to need to develop a few marketing skills.

Speaker B: Yeah, yeah. I mean everyone who kind of moved up to leadership levels and above had, had to kind of do that anyway. You had to have a very good, kind of like a sympathy for the, the other parts of the, all of the things that actually deliver software, whether that's finance or product or marketing or anything. So I think that's going to be the case. And you know, I think the other danger, I think you hear, you hear companies talking about rolling out AI tooling as a strategic advantage. Right. It isn't. It's a strategic advantage that everybody else can instantly have for a Claude subscription. So it's, I think it's working out for companies anyway. Like what, what are you best positioned to do, to do with it? What's your launch platform to add AI to? That's very difficult for somebody else to, to replicate or what's your moat?

Speaker A: Yeah, yeah. It, it can multiply your strengths and also pour oxygen on your fires at the same time.

Speaker B: Right.

Speaker A: It's a multiplier.

Speaker B: Yeah. And you might be kind of lured into a false sense of sort of confidence that you can start competing on other companies terms, you know, or you know, maybe we should start, you know, providing, you know, taxi services through an app. Right. This like, you know, I mean, I think it's going to, I mean you're seeing it a lot as well, you know, a lot of people who suddenly realize they can create apps and they're getting apps onto the app Store and, and, and that's very good but you know, everybody can do that now. Well, you know, I mean there's, there's really a base still a barrier to entry to AI to do that kind of stuff. And, and most people are doing that stuff are still, are still kind of doing it on the side.

Speaker A: Yeah. This reminds me of a kind of pair of questions I want to ask you. I'll ask the first. You must be seeing not traditional coders at work now building things, right?

Speaker B: Yeah, absolutely. Yeah.

Speaker A: How much is that adopted? How well is it going? How much do people over or underestimate their capability?

Speaker B: Yes. Yeah. I mean everybody's building stuff now for sure. And you know, especially in the kind of like the project management, delivery lead ba space. Right. I think for, for their entire career they're limited sort of the limitation on them is the capacity of their delivery team, their software developers. Grab some water. So they've. The best ones I think are kind of, they're pulling in the proof of concept work to their realm and they're probably their better position to do that. The ones who are getting a little bit overconfident are then kind of take, you know, jumping to conclusions and thinking well, why can't we just deploy this? I've done it. Most of them have already realized why that's a bad idea because they then tried to add a second or third feature and the second or third feature came out very wonky and they didn't know what to do about it. And then everything started to fall apart at the seams.

Speaker A: Are we going to have to start developing the skill of adopting vibe coded, well intentioned projects and refactoring them up into something production ready?

Speaker B: Yeah, I think in the, I mean also some of these proof of concepts that they're building are very expensive. Like they set off some agents running over a weekend and you come back and you've spent a couple of thousand dollars, you know, and the business might rightly say, well hold on a minute, I didn't want to spend a couple of thousand dollars on, on that specific idea, you know. Yeah, I don't think that is a reason why people should slow down right now because I think enjoy that while it lasts. But there, there is going to come a point in time where you know, the, the kind of the wire terp technique that I've, I've kind of always used with, with clients before where you know, although being fast is really good, sometimes just taking a little while to, not too long but a little while to think about what you're going to build and being a little bit more accurate and a bit more pointed about exactly what it is actually makes you faster at delivering value over time. So I think there will be a bit of Some kind of funnel that kind of shows that, you know, we still want to be intentional about what we're building and not necessarily just kind of spraying tokens and everything. But right now, you know, it's a brief window of time to do that. So I, I tuck into the token, the token maxim right now.

Speaker A: Yeah, yeah. We've got to develop some skills at using this ability freely before we start to think how we're going to use it. Well, there's always that phase in learning some new skill, right?

Speaker B: Yeah, everyone's just been given chainsaws so, you know, just let them kind of wave it around at their furniture for a little bit and then you go,

Speaker A: what could possibly go wrong? Yeah, eventually they'll learn to chainsaw sculpt.

Speaker B: Yeah, yeah, exactly. Yeah. I mean there'll be a bit of damage along the way and you know, but that's just kind of the way it is.

Speaker A: I think the other question I wanted to ask you, and this is kind of, I don't know if this is the mirror image or if it's, or it ends up being the same question, but that's sort of looking down and across the organization. Have you got any thoughts, experience, tips on how we deal with the way AI is changing the expectations from management?

Speaker B: Yeah, I mean, it's not a new phenomenon. It is something that happened that I used to experience before, especially for startups and scale ups, is you will get someone who produces something that looks very phenomenal very quickly and it used to be limited to, you know, the kind of. There are a certain number, and I've been this person before where we get very interested in solving a problem and we will, our brains will latch onto it and we will work 24 hours a day or as close to that as we can and the weekend, especially when we don't have families, but, and we will produce what looks like an incredible amount of software in a very short space of time and they will mistake that for being, well now I expect that of everybody and, and that's not sustainable. And they learn that now you're, you're going to have a lot. That's just time to buy 20x now, right? You're going to have a lot of people who get, who get the Claude Mania to kind of to bring, bring the whole thing back. They get a lot of people. It's not just techies, it's not just the builders, not just the crafters. These people are getting the Claude Mania as well. You see them online and they are going to produce things that look incredible. And yeah, I think a lot of people need a reminder that to kind of operate things and iterate on it and turn it into something that's sustainable is still a skill and it takes practice and process and rigor. So I think everybody is going to be kind of showing off for a while and you hope that kind of the senior leadership don't start to get overinflated expectations of what actually your workforce can actually take. Yeah. But I think burnout is going to become a real hot topic for the rest of this year and next year. The burnout from people that I think is going to be fueled by these expectations.

Speaker A: Yeah. Sadly so I think you're right and I'm not sure if you have any suggestions on how we can navigate that, but I feel like I don't.

Speaker B: Well, I. I put a hammock up at the bottom of my garden and I kind of force myself to go out there without my phone and lie there staring up at the, you know, the sky for a while. But it's, it's tricky, you know, because it is exciting building stuff and it's. It's easier than ever to build stuff.

Speaker A: Yeah. And it's why we got into this industry for most of us. Right.

Speaker B: Yeah. Yeah. I don't have to decide whether I'm working on Claire like you know, the sort of the clairvoyant agent proximity system or I'm working on my artificial lot. I can do both. In fact, I've got both running behind as we're talking.

Speaker A: Yeah. And they'll probably be able to keep running while you go to that hammock.

Speaker B: Yes, exactly. Yeah. I mean, yeah. I mean it's going to be weird, you know, being single, truly single threaded. It's good. Probably going to be come a term. Right. You know, being out and about just in the moment, not secretly with agents running in the background, you know. I know. I mean I use my phone to check in on them as well.

Speaker A: Yeah, yeah.

Speaker B: You know, I mean we get signal on, on the Northern Line in London now which is great. I'm creating pull requests, you know, through my Claude app on the Northern Line on the Tube.

Speaker A: Yeah, yeah, I've done the same and I think probably on that note it says maybe we should go to the bottom of the garden, lie in the hammock and avoid burning ourselves out for a little while.

Speaker B: But yes, if it wasn't, you know, 33 degrees in the garden, it's too hot for the hammock. But maybe I'll go and lie in front of the air conditioner for a bit.

Speaker A: Yeah, in this weather, it's litter or burnout. But we shall have to see how the future unfolds and try and navigate it as best we can. For now, James Brown, thank you very much for joining me in the present and let's see how we navigate the future.

Speaker B: Thank you very much.

Speaker A: Thank you very much, James. We are all headed into that future together. So, ready or not, here it comes. If you'd like to take a look at James's Claire project, you'll find a link in the show notes. As you may have expected, he's not publishing the Life Simulator yet because there's a chance it's going to head to theme, but if that happens, I'll let you know before you go. If you've enjoyed this episode, please do take a moment to like it, rate it or share it, because it really helps other people find it and if you enjoyed it, they might too. Make sure you're subscribed because we'll be back soon with another episode, but until then, I've been your host, Chris Jenkins. This has been Developer Voices with James Brown. Thanks for listening.
