---
url: https://www.youtube.com/watch?v=0wumMeZsNsI
date_fetched: 2026-11-13
---

# AI is here. Why aren't we shipping faster? (UC Irvine, October 2026)

- **Channel:** Christian Bird
- **URL:** https://www.youtube.com/watch?v=0wumMeZsNsI
- **Duration:** 20m 56s
- **Transcribed:** 2026-11-13

---

So my name's Christian Berg. I'm right now at the Dryden Technology Group. I left Microsoft after 16 years. I wanted to climb another mountain. So now I'm doing IP litigation consulting.

Okay. So the question I'm going to try to address today is AI is here. Why aren't we shipping faster? I'm going to share a number of kind of studies we've together a story of what we have observed at Microsoft. I don't want this story to come off negative.

You're going to see a lot of problems here. For you, that's a great thing. If you're doing research, what are you in search of problems that matter to work on. I'm going to give you a lot of those. So don't read this as everything sucks.

Read this as there's lots of opportunities for growth. All right? Okay. Oh, and full caveat. I hate it when someone sends me an email and it's clear that it was totally AI that did it.

Right. The spine of the problem is this, but the foot gun is here and it's not X, it's Y. So what I appreciate is someone said, lets me know. Oh, I use AI to collect my thoughts, to organize, whatever. You can probably tell right off the bat, I did not draw this.

Okay. So this presentation, I did use AI to, to create it, but the amount of time I put into it is way more than should ever go into a 20 minute talk. So. But yeah, just know that. Okay, So I want to tell you a story about a product at Microsoft.

There's a product that was decades old and one day an engineer was using AI, grabbed a bunch of context and threw it at an agent. It's like, work on this. And it went off, away and came back later that day. The agent had found the problem. Turns out that the last time someone had even attempted fixing this performance problem was a decade ago.

And this is a pro. I can't say the name, but you would recognize it when you saw it. It turns out that when building a tree in memory, it was using a quadratic algorithm. The engineer spotted that, was able to fix it in a day and gave it to management. And it was awesome.

And he said he told our intern who was interviewing him, wow, this is great. I don't think it would always be this easy, but this turned out wonderful. Right? So this is a great story. Okay.

Doesn't always work out that way. A product manager, a different group. This is a story shared with me by a colleague. Saw all of his engineers solving problems with AI and Was like, well, here's this bug on the backlog that really matters. It's been around for a long time.

I'll use AI. And he fixed the bug and he checked it in and he promptly broke production. Now, what's the difference, right? The engineer understood the system. They knew how it worked, they knew what assumptions were baked in, they knew about the dependencies that were in the system.

And so they were able to make sure that the agent knew all that. The PM just said, hey, fix this. And the agent had no idea about any of this information. And so while the changes coming in from engineers was nice and clean, he was just giving it gunk and it broke production for the actually made it all the way to the customer. Right, so that's a problem.

Well, why does this happen? Well, because we all think AI can do anything. A very common story that see with Microsoft is someone shows up with a demo because they're very compelling. And I'm sure anyone that's gone on LinkedIn or Twitter X, whatever it's called today, I can see something where someone vibe coded up an amazing demo. They're like, oh, wow, that's amazing.

And so management sees this and they say, hey, you know, why is it so hard for you guys to do this? This guy on Twitter did it like last night. You've been working on it for a month, you know, and they see, you know, like, my customer wants this right now. Let's do it, Come on. So this is what, this is actually a quote that came from one of the people that Yunha Cha, who's one of the student here, one of my interns over the summer told us.

We get this from our management. It's the worst. And so the puzzle that we have been trying to understand is how do you deal with this influx, all these changes coming in, how do you give them to the customer? So there's a study this year, people looked at 500,000 GitHub developers and they said, it looks like once AI has been introduced, we see the number of commits go up, so increased by 240%. That's huge.

And yet the amount of releases has only gone up 30%. Like that's a big difference. Right? So what's going on with the rest? Where, where did things go?

Well, we tried to understand how things are changing at Microsoft. And one of the things that we asked people, engineers and said, what's, what's different about how you work now that AI is here based on, compared to like years ago, you know, say two, three years ago, when it wasn't. And they what they said, they gave us open responses and we coded them and they said kind of categories. So the first one, and I suspect probably this is true for many of you, I can do what I used to do, only faster. This is not surprising.

What I think was more interesting was I can do more, I can go further, I can do things I wouldn't even attempted before. So one engineer shared a story. He's like, I automated last week in a couple hours. Work that would have taken me 80 hours to do in the past. Now there's no way I could have justified to management spending two weeks, two full time weeks, which if you're also doing meetings, which I hear the same thing about people in meetings, probably the month's worth of engineers work, I never would have been able to justify that.

But now that I was able to automate that in a few hours, I can get that done so I can do stuff I never would have done before. And then there's this new type of work my colleague called it afresh, is that you're working with agents, you're directing them, you're using your judgment. I love that slide. Judgment is so important. You're orchestrating them, you're reviewing it.

This is a whole new type of work. And then people are also going above and beyond just doing work they haven't been assigned to do, but they want to do. Okay, so that's awesome. And what's even better is as people are doing more, they can do. It turns out that writing software is not just limited to engineers.

So Jensen Huang is the CEO of Nvidia. He's the reason that we have AI or maybe you're in this mess, depending on how you can. He said back in 2023, everyone's a programmer now. You just have to say something to a computer, which is awesome and it's super empowering. I listened to a podcast, I think it was like Lenny's podcast.

I know how many you're familiar with it, love it. And there was someone from one of the PMs from OpenAI and they were saying, oh no. I said it was anthropic. They said, it's so empowering, the designers can write the code and I don't have to wait for engineers. And that's awesome.

And so now you have not just code coming from engineers, but you have, you also have product managers and designers and other teams. But it turns out that what they're shipping is not as clean and positive as what the engineers do. And maybe the fact that everyone can write code is not the best situation. This is another quote from someone that I interviewed. He said, I see code that is under my management changed by people I've never heard of before.

And I wasn't even included on the PR for it. So someone I don't know submitted a change, someone I don't know reviewed the PR for code that I own and then it caused problems, right? That's not a good situation. And so it turns out that writing code is just a part of it, right? One person said, if you can do the code in one day, you're still talking 30 days before it actually gets deployed.

Now why is that? Well, there's a whole ton of other stuff that happens, right? You write the code, that's fine. Someone has to review it, someone has to test it, you have to integrate it. Integration is interesting.

One person that we talked to said, okay, we have a product that's literally 40 years old, has millions of lines of code. Writing the feature is not the hard part. Integrating that feature into the millions of lines of code that are 40 years old, that is the hard part. And I said, he quote, agents suck at it. Right?

And I think you've probably experienced this. I mean, how many people have five Coded something from scratch to, to scratch an itch, right? It's amazing. You feel like, you literally feel like there's power in these hands. It's so awesome.

I have lost way too many hours of sleep at night building something because it's so empowering, exciting. But how often was that built, like from scratch, from nothing? Most of the time for me it's. I start with an empty file and it just starts building. How often do you start with an old crafty million line piece of code?

It's not very often. And how many times you see demos from Anthropic or OpenAI that work on a pre existing system? Never. They're like, here's a picture. Build me this UI from scratch.

Well, sure, it can do that, but most engineers are not working in those situations, right? There's other things going on. So one person said, if we used to take a year to get a feature done, that we can now do in a month, that's awesome. No shade on that, but in some sense a year's worth of collaboration has to happen in that month. That's really important.

Writing the code is just a piece. There's all this other stuff going on, right? And so what happens is you have all these other pieces that have to happen. This is great, right? We've got all these changes coming in.

This is clean water. These are high quality changes. That's wonderful. Engineers are being able to use AI to generate code in a great way. But shipping the code is all the things that have to happen along the way.

And as you see, things have become just to spill out. Right. We can't keep up. And part of the reason is all these are quotes from our study, from various studies that we've done. Someone said the people qualified to do the shipping work, they didn't multiply alongside this increase in the ability to generate code.

And so because of that, the humans are the bottleneck. Now is that good or is that bad? We're not sure. Here's the problems where you can tackle because they're open. So one pace at which we have to review is fast outpacing our ability to review.

And we've heard this. Anyone that's studying, you know, if you go on any programming subreddit like for five minutes, you're going to hear this problem, right? He said, if I'm reviewing, I feel like maybe I'm not a team player. Understandable. He's a senior dev.

Okay, so I'll focus on all my reviewing and I'm reviewing as fast as I can. But now my stuff's slipping. So he's trying to do the job of everybody. And if you have a thousand morning, which that top, I've never heard it that bad like, so he's trying to review it. It's just not going to work, right.

We don't have enough people, they don't have the skills. And so along the way management's expectation has increased. Right. And so the problem is that people, they can't keep up. And so the work just falls on the floor and we have these long backlogs of PRs, bugs that need to be fixed and people are working as hard as they can and you have this kind of organic, even unsaid expectation.

So I don't think management at Microsoft is bad and I don't think that they're different from any other large organizational company. But it's like everything's speeding up and there's kind of this implicit pressure that people feel, maybe even they put on themselves that we gotta go faster, we gotta go faster. You know what happens when you start going faster? You forget details and stuff slips through and it just kind of goes down. That's not great.

So Yunha called this systemic conversion string. So she called the process the path from code sitting in a repository to make you a customer all that stuff. Conversion, the cost of generating code fell, but the cost of shipping it barely moved. And expectations for management treated the same. If we can write code twice as fast, why can't we integrate twice as fast?

Why can't we review twice as fast? We have AI. Aren't models going to save us? Well, turns out maybe not quite yet. We did a study over the summer where we actually looked at how well do models review code.

We took code changes had been reviewed at Microsoft and also in open source. And we looked at what were the issues that people found during review. And also were the issues that they didn't find that showed up as bugs later that we wish would have been caught. We found that the models are getting better. So when you compare.

So this is frontier models. So like Opus, I think it's time for 7, 4, 8, February, 5, 5 in July, NGBT, right. And Gemini, all the frontier models, it got better in terms of how many they caught. But still, if you're letting 30% of the things through, that's a problem. You can add humans to the mix.

That gets you a little bit closer. Interestingly, they find different types of things. So people are really good at finding maintainability problems. The models are not as good at that. They focus more on severe, severe problems and reliability bugs for all of these.

And I'm happy to share out the slides. You probably can't load the things on the front so you can't see the papers, but that is, I'm sure we'll get into fse unless someone uses AI to review the paper. What? Yeah, that's right, I did. So we'll see.

But the goal is at Microsoft, they want to have what they call lights off software factories. As in you don't have to turn on the lights because there are no humans there. And what we found was you can do that, but only for narrowly scoped and low consequence pr. If it doesn't matter, let a model do it. If it does matter, don't let a model do it.

Okay, so but we could, right? We could have a model, we could bypass the human review and we could say let AI do it. And it turns out we actually asked developers and Marco, where are you? But Marco helped out in this study. We wanted to ask developers, what are you willing to give to AI and just.

And let it do its thing and give it autonomy. And what do you want to still hold back and hold on to yourself? Because we want to know where this frontier is. What types of tasks can we just automate and what type of tasks do engineers want to hold on to? There were two findings.

One was really was not surprising at all. One I found surprising. The first one was. So we asked the answer, why would they hold it back and not let AI do it completely? The first one is accountability.

If my butt's on the line, then I want to make sure that I see what's going on. And so, because I don't want to be called up at 3am because the system went down. So they may use AI, but they are going to hold. They're going to hold the switch in their hand and they're going to pull the switch. They're not going to let AI just commit into the repository.

That's not too surprising. The more interesting thing was that they said, this is my identity. So it's like, I'm a developer, I really like writing code. This is what I went to school for. This is what I do on my free time.

I don't want AI to take away the part of that defines who I am. Again, you found this too. It sounds like the people are like, this defines this is who I am, right? This is what I went to school for. They didn't like that.

But the interesting thing is they have this workload, this backup. And one person said, see, I'm probably quote exactly right, but he's like, I really want to do this because it's what brings me joy and it's my identity, but I simply can't keep up. And so I'm going to have to let AI do it. So these are these competing pressures, right? And this is what I think a lot of companies are grappling with, is how much do I let through, how do I mitigate the potential risks, how do I prepare for the failures?

Because there will be failures. It's going to happen. How do we prepare for that? One interesting thing, because there's all this pressure, is that it actually stops people's ability to learn. So we did this study where we actually found 27 pairs of developers who want to know, why do some people use AI and some people do not?

And each pair, we found pairs of developers in a team. So they were all working on the same code base, had the same manager, the same culture and same location. And we identified the person that used AI the least and the person that used AI the most and tried to understand what's going on. And so, like, the median over, I think 56 days of telemetry, they collected the low user, used AI on 5.5 days, the high user, the median was 33 days. And we learned a whole bunch of things.

But what I think is relevant for this talk is what Courtney Miller called the productivity pressure paradox. What she found was that some people feel so much pressure to deliver, they don't have the time to learn to use their tools. So the way she wrote it, developers lack the time necessary to develop the skills that would save time. So management is pressing on so hard. I can't.

You know, riding a bike seems really hard. I'm not going to do that. I'm going to continue to walk. It's like, no, if you took the time, it would get better. But they just feel like they don't.

They can't do that. So what are things that we can do? Well, one, I think we've been measuring the right thing. We've been measuring over here, lines of code and maybe PRs, but really we should measure the outcomes that matter and not the level of activity. Now, that's super easy for me to say up here, right?

What are those outcomes? There's tons of different outcomes. It's really hard to measure, right. This value. In fact, I met with some students this morning, and one said, how should we be measuring productivity now in this world of AI, does it need to change?

And our solution was like, our answer was that we've never been able to measure productivity very well at all. We've always measured over on the left. It's really hard to measure on the right. But to the degree that we can, we should be focusing on the outcomes that we care about. And especially if we're saying, like, AI is helping us to do things faster, we should not focus on code produced, but outcomes that we care about next.

Verification, I think, has become much, much more important because we're not generating, but we do need to verify, and so we need to scale that up. So that may mean more people, it may be better tooling. There's a whole host of things that we could think about to help people be more effective with verifying code. Review today has not changed a whole lot since, like the 70s. You see a diff, you look over it, you make some comments, right, that it hasn't changed a whole lot.

So I think there's a whole host of things we could try in terms of allowing developers to review code. And I'm happy to chat with anyone about ideas I have, and I'm sure a lot of you have ideas like how do you help developers scale up the verification without giving up quality? Right? And then protect the people who are helping to ship things if you gave them time to learn their tools, they would stop running up stairs with buckets. They'd learn to use those tools.

They'd be more effective because they had the time to do it and then have the pressure of the day to day to protect them. All the people in this pipeline, they're all really, really smart and they're trying to do the best thing, but in some cases they're in an impossible situation. And I think you are all in situations where you can try to tackle these problems, understand what they're doing, and you have the time to explore and to experiment and to propose things that they're probably dying to try. So. All right, so this is one of, this is just kind of an ending in one of our papers that I think I thought kind of profound that we have a choice to set the boundaries deliberately through how roles, workflows and jobs are designed, or we can let the tools, the tool default, reset them with each release.

So do we let the tools dictate to us how we use them or can we define them and what does that need to look like? And this is, it's really kind of a pithy statement in a paper, but I think it's really profound if you think about it. It's not easy. None of these things are easy. But like, that's why we want to tackle them, right?

That's what makes them interesting. I think they're really thorny. So to call back to the PM who submitted the breaking change. They said, I now have a new respect for my colleagues who are engineers. Maybe I should stay in my lane.

And I thought that was encouraging. Engineering is a skill, it's a talent. And the value of it and the judgment that goes into it has not gone away. I don't think it will. It, it will shift and you need to pivot, as many people have said today.

But it's not going away. I wouldn't be worried about that. So take away. Why do I think we're not shipping faster now? Well, one is there's more work coming from more people, some of them more qualified than others.

The people who ship it did not multiply. Right. They're limited and they're not willing to just hand over everything to AI. I would posit that's a good thing. It's our job to help them be able to scale and we count what goes in.

We should be focusing more on what comes out. So that's my hell of a overview. I'm happy to share these slides with the links to all the papers and things. I think it's a really exciting time to be studying software engineering because there's so much going on, but there's so much help that is needed and you're in a prime place to explore it. So that's my talk.

I'm happy to take any questions. I really appreciate the invite here.
