---
url: https://gist.github.com/a394507d48e3135a19de8cb3a50a369f
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Adoption is near-universal, but organizational transformation is rare. 92.6% of developers use an AI coding assistant at least monthly, yet an MIT study of 152 orgs found “high adoption, low transformation.” Using the tool does not equal impact.

AI is an accelerator, not a fixer. It amplifies existing organizational health: functional teams get faster with higher quality; dysfunctional teams become “dysfunctional faster” and can see twice as many customer-facing incidents.

Time savings have plateaued around 10%. Self-reported hours saved hover at roughly 4 hours per developer per week, a figure that has barely moved in recent quarters. The ceiling on individual coding-task productivity is low.

AI-authored code in production is rising fast. Industry-wide, 26.9% of merged code was written by AI without significant human intervention, up from 22% the prior quarter. Daily AI users exceed 30%.

Onboarding time has been cut in half. Time-to-10th-PR has dropped by 50% since Q1 2024, and Microsoft research shows that faster onboarding performance sticks with an engineer for their first two years.

Agentic workflows are the expanding frontier. Over 50% of developers at instrumented companies use agentic workflows daily. The ceiling is much higher, but the organizational challenges are the same.

Winning orgs do three things: they set concrete goals and measure progress against them; they treat developer experience (feedback loops, docs, CI) as critical infrastructure for AI; and they focus experimentation on real customer problems, not moonshots.

2. Pithy and provocative quotes

“Adoption doesn't mean impact. Using the tool doesn't mean that it's gonna actually advance your organization or do anything. It is an organizational problem that needs organizational change management.” — Framing the core disconnect between tool usage and business results.

“AI is an accelerator, it's a multiplier, and it is moving organizations off in different directions. … Some organizations are facing twice as many customer facing incidents … At the same time, companies are also experiencing 50% fewer incidents.” — The uneven, bimodal effect of AI on quality.

“Organizations who were dysfunctional already — now they're more dysfunctional, they're dysfunctional faster.” — A blunt summary of AI amplifying existing problems.

“Spray and pray does not work. … Just giving all of your developers licenses and hoping for the best. It does not work. I can say that very, very clearly.” — Rejecting the default enterprise AI rollout strategy.

“Just anything that you were going to talk about with your leadership team about developer experience — just call it agent experience and you'll get money for it. … It is disheartening that we didn't want to spend the money when it came to human engineers. But when it comes to robot engineers, we're okay with it.” — A cynical but practical observation on how to fund long-neglected DevEx.

“The risk is if we don't address the systems level problems, we will just take them to space with us.” — The core warning about applying AI without fixing underlying organizational issues.

“Stay grounded, stay skeptical, stay human. Most of all, stay pragmatic.” — The closing call to balance wonder with reality.

3. Tools, practices, and methodologies

AI Measurement Framework (co-authored with Abi Noda / DX): A framework that complements the Core 4 by tracking not just AI usage/adoption but translating it into organizational impact (speed, DevEx, quality, innovation ratio) and cost. Use it to connect adoption to real outcomes instead of just counting licenses.

DORA AI Capabilities Model (dora.dev): An AI readiness model backed by DORA’s research data. It identifies practices correlated with good AI outcomes — e.g., having a clear, communicated AI stance. Use it to audit organizational readiness and convince leadership.

ThoughtWorks Forest Framework (thoughtworks.com white papers): Another industry-backed AI readiness model, similar in spirit to the DORA model. Use it for internal audits on whether the org is doing the right things to benefit from AI experimentation.

Time-to-10th-PR as an onboarding metric: The industry-aligned milestone for developer onboarding. The talk shows it has halved with AI usage and that Microsoft research links faster time-to-10th-PR to sustained productivity gains for two years. Use it to measure and justify AI-assisted onboarding.

Multi-agent consensus patterns (JPMorgan Chase’s MAFA framework): A multi-agent workflow where specialized agents annotate interactions, a second set re-ranks and validates output, and consensus algorithms resolve disagreements. The speaker predicts “consensus among agents will be a huge problem to solve in 2026.”

RALPH loops for rapid prototyping (Haven Headache Center example): Using agentic workflows that take Linear and Figma artifacts, convert them into a PRD and JSON, then run RALPH loops to generate high-quality prototypes with documentation and tests. Use for disposable, high-quality prototypes faster than manual building.

HIPAA-compliant model trained on symptom logs (Haven example): Training a compliant model on hundreds of thousands of patient symptom messages to route them to medication refills or follow-ups. Use to meet users where they are and improve clinical outcomes.

4. Unanswered questions and omissions

What does the “low transformation” org actually do to change? The talk identifies the problem — adoption without impact — and says orgs need change management, but it doesn’t outline what that change management looks like in practice. What specific steps move a company from the 92.6% adoption bucket into the “winning” bucket?

The cost question is raised and immediately dropped. The AI Measurement Framework includes a cost dimension (“are we getting a good deal?”), and the speaker notes costs keep going up, but there’s no data or guidance on how to evaluate ROI when model and inference costs are rising. The economic sustainability tension is acknowledged but unresolved.

The “agent experience” funding hack is cynical but unexplored. The speaker suggests rebranding DevEx initiatives as “agent experience” to unlock budget, but doesn’t address the ethical or long-term consequences of funding human infrastructure only when it serves AI. Is this a sustainable strategy or a short-term trick?

What happens to the developers who aren’t daily AI users? The data shows a gap between 92.6% monthly adoption and 75% weekly adoption, and daily users cresting 30% AI-authored code. The talk doesn’t explore the experience or productivity of the non-daily, non-power users — are they being left behind?

No discussion of non-engineering functions. The entire dataset and argument focus on developers and code. How do these patterns apply to AI adoption in product, design, marketing, or operations? The organizational transformation argument implies cross-functional change, but the evidence is entirely engineering-centric.

The environmental impact is name-checked and ignored. The speaker mentions “skepticism about the real economic impact of AI, given how expensive it is, given the environmental impact,” but never returns to the environmental dimension. For a talk urging pragmatism, this is a conspicuous omission.

What’s the counterargument to “just experiment on customer problems”? The talk says moonshot experimentation is fine but unsustainable for the whole org, and that real wins come from solving customer problems. But it doesn’t address the tension: many breakthrough innovations (including space program spinoffs) came from exploration that wasn’t directly tied to immediate customer problems. How do orgs balance the portfolio?

Today I wanted to have a really pragmatic and down to earth conversation about AI, what is actually happening in our organizations, what you can expect to happen, and how agents are changing the game. I thought in order to have this really pragmatic down to earth conversation, I wanted to take us to space. I do see a lot of parallels between the age of exploration and the space race and the age of AI. So last week I was talking with a cto, co founder of a small startup. I was also talking with a principal engineering lead at a very big bank, highly regulated, and we sat for a solid 15 minutes talking about all the cool stuff that we were building, how it's brought back, the joy of coding.

And we just had all of these ideas and it seems like we couldn't build fast enough. There is so much to learn and so much to build. And it reminds me of this really lovely quote from Carl Sagan that I love, that somewhere something incredible is waiting to be known. And I think that this quote really captures what a lot of us are feeling about AI and about the experimentation and just the possibility that is out there. This is the same feeling that we had with the age of space exploration.

Going to the moon, going to Mars, but it didn't come without skepticism. Why spend all of this money experimenting and going to the moon when we had lots of problems to solve here on Earth? We had a lot of wonder, but we also had a lot of skepticism because space exploration wasn't just about science, it was also global, it was economic, it was political. Space wasn't a silver bullet to solve all of the problems that we had with humanity. But we also can't deny that when a man landed on the moon, that it was a very pivotal, defining moment for all of humankind and had a sense of wonder and had the world in awe.

It was about redefining what was possible. And similarly, we have a lot of wonder and a lot of optimism and a lot of promise about AI. We can talk about productivity boosts and all of the hype around 100% productivity boost in all of our code being written by AI. We have the promise of perhaps the first single person billion dollar startup with a person with an idea and an army of agents. There's a lot of optimism out there.

Similarly, there's a lot of skepticism. There's a lot of skepticism in the corporate world about the real economic impact of AI, given how expensive it is, given the environmental impact. There's also a lot of skepticism and a lot of different studies about the real productivity impact. In certain circumstances it can be really, it can really accelerate, and in other circumstances it can actually slow us down and get in our way. It's hard to know what's real, but as technology changes, it is really good and fine to have that sense of wonder of exploring the universe while also realizing that we have problems here on Earth to solve.

We have to learn how to balance that sense of wonder and curiosity with the acknowledgement that we are living in reality and we need to keep our feet firmly planted on this Earth. We need to understand how these experiments are actually going to apply to everyday companies. How are we actually going to improve the world around us? We need to keep the sense of wonder while also balancing it with pragmatism and beating the hype by looking at data. And so that's what I want to do right now.

I'm going to share some brand new AI industry benchmarks with you. This is new data that no one has ever seen ever before in the world. Okay, it's coming right now to you. I just pulled these down. I guess the statsig team has seen them because they saw the preview of my slides.

But aside from them, no one has ever seen it. This is though, not really surprising because a lot of these numbers have not changed very much from the last quarter. So what we're looking at here is a sample of 121,000 developers at over 450 companies. This data was pulled from November through February 1, 2026. I really just did this.

We're sitting around 92.6 of developers are using an AI coding assistant at least once a month to get their work done. And about 75% of developers are using an AI coding assistant at least once a week. When I say AI coding assistant, most developers define that as cursor, codex, copilot, Claude, not chatgpt necessarily. But it is a bit open ended. So keep that in mind when it comes to time savings.

Time savings is not the only measure of productivity impact, but it is an important signal. It's a good leading indicator. We're sitting around 4.08 self reported hours saved due to AI tool usage per week per developer. This is not all too different from the number that came in Q2 of 2025 and then the number for Q4 of 2025 was about 3.6 or 7. So this is kind of hovering around the four hour mark.

And there's been a few articles for example from Google in the last year citing about a 10% productivity increase. And if we look at it in Terms of time savings, we're kind of hovering around that 10% mark. It hasn't changed dramatically over the last few quarters. What is changing and what is moving up very quickly is the amount of code getting merged upstream or in a customer facing environment that was written by AI that was merged without significant human intervention. We call that AI authored code.

And in a sample of around 42,600 developers from that same timeframe November 1st to February 1st, 2026, we're at about 26.9% industry wide. For all of these developers, that's how much code is hitting production. That was AI authored. This is moving up from 22% in the last quarter, which is actually a pretty significant change quarter over quarter. And we can see that daily users of AI have crested over that 30% mark.

So almost a third of their code is being written by AI that is actually being merged, passing through code review and getting into a customer facing environment. One of my favorite use cases for applying AI is to onboarding. And I had a bit of a hunch that AI was going to be a great tool for onboarding, helping connect people with information earlier and sooner. And I have all of this data and I thought, let me look at this quarter over quarter and in fact if we look at Q1 of 2024 all the way over here on the left side and Fast forward to Q4 of 2025, we have about a half, we have a half reduction in onboarding time. This is looking at the time to 10th PR.

So by the time a developer hits their 10th PR, that's a pretty important onboarding milestone that the industry has mostly aligned on in terms of onboarding. And that has been cut in half now. And when we correlate that with the uptick of AI usage, it makes a really pretty graph. AI is fantastic for onboarding and this is not just brand new hires to your company. We've also seen plenty of evidence that this is for engineers who are moving projects or even non engineers coming onboarding into projects.

What's really important about this number is that there was a separate study done by Brian Hauk at Microsoft, he's the co author of the Space Framework of Developer Productivity. And they found in Microsoft's context that the time to 10th PR actually that performance sticks with an engineer for their first two years of tenure. So if you onboard faster, that productivity gain isn't just onboarding, it actually sticks with them for at least two years after they have started at the company. So this is a very important and significant trend that we're seeing here with using AI to connect developers, reduce cognitive load, and get them onboarded more quickly into their code bases. One thing that's really important for me to call out, although I have just shared with you, industry benchmarks and averages are just math.

And as the polls move further away from each other, the average stays the same. Average does not mean typical, it does not mean what is going to happen to you, and it doesn't mean what a common experience is. One thing that is absolutely true, one thing that is common, is that there is no typical experience with AI. There is no typical experience with AI. It is extremely different in every single company because every company has their own problems and their own culture.

This uneven impact can take us back to space for just a minute, so we can go back to the origins of the universe. We had the Big bang and there was this massive release of energy. And as this energy released, the space and time in between objects grows bigger. Things are moving apart. And for a lot of us, the emergence of AI and AI coming into our organizations and in the industry has felt a lot like this big bang.

We've had this explosive release of energy in the center of our world and things keep moving apart. Organizational performance is multidimensional and these organizations are just going off into different extremes based on what they were doing before. AI is an accelerator, it's a multiplier, and it is moving organizations off in different directions. The best example I can share with you of this is quality. Okay, so in this case, this is not every organization, but some organizations are facing twice as many customer facing incidents.

And this is from a sample of over 67,000 developers from Q1. So that same time frame of November to February. So just looking in that timeframe, organizations are experiencing twice as many customer facing incidents at the same time. At the same time, companies are also experiencing 50% fewer incidents. So some companies have used AI.

They have a really healthy system. It has amplified that system. They are seeing fewer incidents, they're moving faster, they are accelerating with higher quality, higher code maintainability, higher change competence. On the other side though, blasting off into the other part of the universe, we have organizations who were dysfunctional already. No, they're more dysfunctional, they're dysfunctional faster.

Okay. Similarly to this uneven impact, organizations are seeing really uneven results. Like economically from using AI. There are a lot of steep drop offs when it comes to using AI in a pilot context to production, and then actually trying to tie it to profit. This is from an MIT study that was published in July of 2025, called the Gen AI divide.

And what this study concluded, they did a survey of 152 organizations, was that right now where we are in the industry is that we have really high adoption, that 92.6 number. Dora also does its own research. We're hovering around that 90% adoption number. High adoption, but actually low transformation. Because as it turns out, transformation is really uncomfortable.

And organizations that were ready to give up on the cloud transformation, on the agile transformation, are also giving up on their AI transformations. It is really, really difficult to look at your whole organization and look at the problems and think, hmm, we gotta change something about this. And that is what organizations need to do in order to actually see change to their bottom line. All of this to say back to my previous point, we have 92.6% adoption among developers in our industry, but adoption doesn't mean impact. Using the tool doesn't mean that it's gonna actually advance your organization or do anything.

It is an organizational problem that needs organizational change management. But that's not really what we were promised with. All of the hype was like, hey, experiment with AI and then something happens and then we profit. What happens, though, is that these tools were primarily deployed into individual coding tasks. And what this MIT study found in this high adoption load transformation is that when we apply it only to the surface area of a developer sitting at their desk, there is a very, very low ceiling of productivity gain.

This is an organizational problem. If we want organizational results, we have to think about it on an organizational level, not on a coding task level. Fortunately, our universe is expanding right now, and that expanding is coming through the use of agents in agentic workflows. Our universe is getting bigger, and so are all of the promises and all of the hype, but so is the possibility. So let's go back to the moon landing, right?

Like, the ultimate hype was that we're all going to be living on the moon by now in flying cars, like Jetson style. Similarly, here we have a little bit of crazy ideas. Gastown, if any of you have used it, there's just like, there's so much crazy stuff to do right now. Gastown is infinitely interesting to me. There are so many interesting things.

Disclaimer. Don't use Gastown. It is unhinged. We've got openclaw, Moltbot, cloudbot, whatever it's called. We've got RALPH Loops, we've got all the stuff, right?

There is so much experimentation and so much fun. It's just really fun to build. But me building my nail polish matching color scheme app while I'm sitting at the nail salon is not the same as a multinational bank being able to change their revenue because of AI. Those are really different things. And I was at this retreat with Martin Fowler and Kent, who are, I think, back there.

Hello. We'll talk about that a bit more later. We spent a lot of time trying to connect AI and the use of AI to bottom line, to profit, to P and L. And interestingly, kind of where we landed at the end was this question of what is the value of innovation? Was it still valuable to go to the moon even though I'm not really located on the moon right now?

And I would argue that yes, it is valuable to innovate and that can get into some murky area because this is a business, right? This isn't just society and doing things for the good of humankind. We have to do them in an economic context. And that can get a little bit tricky. So when we think about this quote, something, somewhere, something incredible is waiting to be known.

There is a sense of wonder. And AI and space are both the age of exploration and it is so exciting. But the point of going to the moon wasn't that we all need to live on the moon. In fact, the point of going to the moon and the point of exploring and doing all this crazy stuff was to improve life on Earth. It was to use the space exploration and all of this wonder to apply it to the systems level problems that we had back on Earth.

Not everyone wants to live on the moon, but we have sunglasses, we have space blankets, we have barcodes, we have quartz watches. We have so much technology and so many improvements back on Earth because of this crazy age of exploration where we all went to space. Even though we're not living on the moon, we've still used the lessons and applied it to our systems back here on Earth. And so thinking about agentic workflows, agents expand the possibilities of what we can build, how we can build it and who we can build it for. Not everyone goes to the moon and it's okay not to go to the moon.

Not everyone is going to be building crazy stuff with Gastown every day in your enterprise context. And that's also okay because the experimentation helps push the boundary of what's possible and helps us think about solving problems in new ways. So let's talk a little bit about how agents are being used in the industry right now. Again, this is new data that I'm sharing for the first time here. Agentic use is on the Rise.

There's not a lot of companies, honestly, that are so far ahead of the curve that they're already instrumenting their agentic use cases with really good telemetry. The sample is a little bit smaller. It's around 3,000 developers at six companies. Keep in mind these companies are ahead of the curve. They're already instrumenting their agentic workflows with telemetry.

We have about 80% of developers using these agentic workflows at least once a week, with over 50% using agentic workflows every single day to get their work done. We talked about Codex, I think in the previous Panel. So on February 2nd, the Codex desktop app was released and since then there's been over a million downloads by now. I got this data yesterday, I'm sure it's quite different by now. There's been a 60% growth in users just in the last week.

They also launched GPT 5.3 codecs last Thursday. They're processing trillions of tokens per week internally. At OpenAI, 95% of developers are using Codex to ship stuff. And of the developers who are using Codex versus other AI tools, the developers who use Codex are shipping about 60% more PRs per week, which is very interesting. A data point.

Not the only data point, but it just speaks to the very high ceiling, the high possibility, the sense of wonder that we have with building all of this stuff with cool new tools like agentic workflows. I want to bring it back to a non AI startup though. I want to highlight Haven Headache and Migraine Center. So this is a company that's based here in San Francisco, actually just a few blocks away. Haven set out to answer the question, can we solve headaches with zoom?

And it turns out you can. So if you're a headache sufferer, this might be useful for you to learn about. In healthcare, it's really, really crucial for Haven and their development team to distinguish between using agents for durable code or disposable code. One of the things that they're doing that's very cool since they are a disruptor, they are a small startup is using agentic workflows to rapidly prototype new custom like new patient workflows. So they're working on a patient portal building with RALPH loops, taking linear and figma artifacts, changing it into a prd, spitting that out in JSON and then just having RALPH loops run.

What they're getting though isn't garbage disposable AI slop. What they're getting is really high quality prototypes with really excellent Documentation, excellent tests, much higher quality at a way faster rate than they would have if they would have built it by hand the old fashioned way. The other thing that they're doing that I really admire is improving the standard of care for their patients by training a HIPAA compliant model on hundreds of thousands of symptom logs. So Haven meets you where you're at. You get a text message, you can log your symptoms and then they can instrument your care, figure out what needs to happen from there.

So they're training a HIPAA compliant model on hundreds of thousands of these messages so that those messages can be routed to medication refill or schedule, follow up appointment. Just meets you where you are. And the result of this is that they have 3x, the industry average in customer satisfaction for a healthcare tool like this, but also real meaningful clinical outcomes. So their patients have fewer headache days per month and also the severity of their headaches is much less severe. So good job Haven.

In the enterprise, there are lots of examples of big enterprise companies experimenting with agent workflows. So there's an enterprise manufacturing company that's using it for solely internal developer purposes. They used Copilot and Claude to build out a dev portal to accelerate developer onboarding. At Cisco, there's 18,000 engineers using codecs daily. They're using the codecs for complex migrations and also code review, leading to a 50% reduction in the amount of time it takes to do code review.

There's a really cool paper as well by JPMorgan Chase's Multi Agent Framework for annotation MAFA. If you Google that, you can find the source paper. It's really fascinating. What they're doing is building out a whole business of agents. So a true multi agent workflow similar to Gastown, where each agent has a special job to do.

What they're also doing in this model is introducing consensus among the agents. So they're taking all of these interactions and then they're annotating them. This was the intent, what was it? An faq? What were all of these interactions?

The agents are annotating them. And then there's another set of agents who are responsible for re ranking and calibrating and validating the output. And then of course we have to introduce consensus algorithms to the party because now we have multiple agents with maybe multiple different opinions about things. This is really fascinating and I believe consensus among agents is going to be a huge problem to solve in 2026. I spoke about this retreat.

I was lucky enough to be invited by Martin Fowler and ThoughtWorks to the Future of Software Development retreat celebrating the 25th anniversary of the Agile Manifesto, Gerge joined me. A few other folks who are here also joined me. We spent a day and a half up in the mountains talking about agents. That's really all we talked about, about using agents responsibly, ethically, sustainably, how we can use them for organizations. And our conclusion, even though there was so much interesting stuff, Steve Yegi was there, we were working on Gastown.

Things like there was a lot of experimentation happening, but the conclusion that we came to was that AI does not solve organizational systems problems. It only can do that when you apply AI to the system problem, which means you need to acknowledge that the system problem exists in the first place. AI is not a magic silver bul. Even though things like Gastown exist, even though there is so much sense of curiosity and wonder in the universe, we kind of had a sort of off the cuff conversation. Kent Beck, Steve and I were just catching up outside of one of the sessions, in between conversations, and here's sort of where we summarize our thoughts.

Organizations are constrained by human and systems level problems. We remain skeptical of the promise of any technology to improve organizational performance without first addressing those human and systems level constraints. We remain skeptical and we also remain human. Because the risk is if we don't address the systems level problems, we will just take them to space with us. We will just take them to space with us.

We're not actually going to solve the human factors that are the driving force behind all of the constraints that organizations have right now. We can apply AI to those problems, but we still need to solve them. We can't just go to the moon and expect that pollution and garbage and traffic aren't going to be a problem anymore. And so the question is not how to colonize Mars, but the question is how to get real organizational impact with agents and AI. At this retreat, we also talked a lot about common factors that we see.

What do we see organizations doing? What are the common patterns? That is kind of like the secret to winning. What do they have in common? The first one is that organizations who win with AI and are winning with AI have goals and they measure their progress against those goals.

Spray and pray does not work. Spray and pray. What I mean by that is just giving all of your developers licenses and hoping for the best. It does not work. I can say that very, very clearly.

I have a lot of evidence that does not work. If you can point AI innovation and that experimentation to a problem, have a concrete goal, and then measure if you're reaching that goal. That is what winning organizations are doing right now. Because as Spock has told us, insufficient facts always invite danger. We need to measure things, we need to have data.

And I know this is something that's really difficult for a lot of organizations right now because developer productivity and engineering excellence are also really hard problems. And this is happening all at the intersection. So I have something that can help you if that is in a problem that you're facing in your organization. This is the AI Measurement Framework. This is a framework that I co authored with abhinoda, who's the CEO of DX.

This complements our Core 4 framework, which some of you might have heard otherwise. It's in the impact column here. What we're looking to do is track not just usage and adoption and utilization of AI, but then also translate that into real organizational impact. Is this changing your speed, your developer experience, your quality, your innovation ratio? Those are really important questions to connect the adoption to impact.

Finally, we have to look at the cost. Are we getting a good deal? Maybe some of us are for now. And we need to understand as the cost of these tools keeps going up and up, is the investment the right one? The second thing that is helping organizations win is that developer experience matters now more than ever.

Here is a piece of very unconventional advice that I will give you. Is just anything that you were going to talk about with your leadership team about developer experience. Just call it agent experience and you'll get money for it. It's funny, but it works. It works because developer experience, feedback loops, clearly defined services, great documentation, fast CI.

These are all things that we have been screaming about for decades, literally. And we've been begging for pennies from our organizations to please let us invest, please let us invest in developer experience. And we've been told no over and over again. Come to find out. In fact, these are the things that make AI really successful.

We need to have really solid testing and quality practices. We need to have great documentation. These are critical for agentic workflows. It is disheartening that we didn't want to spend the money when it came to human engineers. But when it comes to robot engineers, we're okay with it.

But that is the world that we live in and let's capitalize on our opportunity. So devex matters more than ever. In fact, when we look at the data right now, remember we're hovering around that four hour mark for time savings. When we look at all of the other factors of developer experience, AI time savings is not going to make up for bad meeting, like bad meeting culture and lots of interruptions and developers who are constantly being pulled out of their work, unplanned work interruptions, outages, those kinds of things. AI will not make up for that.

We can use AI to help solve that problem, but AI in and of itself is not going to make up for it. Then when we look kind of in the bottom half, build and test wait time, toil and dev environment, we put all that together. We realize that just the time savings from coding tasks speed up isn't going to get us very far. But what will get us far is when we can take AI and point it at those problems. Can we use AI to help reduce meeting frequency?

Can we use AI to improve CI wait time? Can we use AI to reduce dev environment toil? That is what winning organizations are doing right now. They are putting devex at the center of their universe and seeing AI as a tool to fix systems level problems. They're doing it also on an organizational level.

If you want organizational outcomes like revenue, P and L time to market, you have to think about AI as an organizational problem, not as an individual problem that your developer needs to solve at their desk. It has to apply to workflows that span entire value streams. Back to that MIT study. When we looked at the barriers to organizational adoption or the organizational barriers to AI adoption, they weren't technical. This wasn't about the models necessarily.

It wasn't even about the tools that wrap the models. It was about things like change management or lack of executive sponsorship. When you have an executive team saying go, go, go with AI, but they themselves have never cracked their laptop open and fired up Windsurf or cloud code or codux. Poor user experience, just very unclear expectations about AI. Those are the things that get in the way.

If this sounds familiar to you and perhaps your organization could do a better job, there's two things that I want to point you to. The first one is the DORA AI Capabilities model. These are models that kind of communicate and help you get ready for AI. So think about this as an AI readiness model and AI capabilities model. This has a crazy amount of data from organizations that DORA studies.

They do a lot more than just the four key DORA metrics. Finding correlations between practices that organizations have and good outcomes with AI. So if you use AI and have a good clear and communicated AI stance, you are going to do better organizationally than a company that does not have one. You can find this@dora.dev. it's the Dora AI capabilities model.

There was just A new paper that came out last month. Last month, Nathan is here who leads Dora over at Google Cloud. If you want to talk to him about this, he's probably the the guy. The other One is the ThoughtWorks Forest framework. This is similar to the AI capabilities model.

Kind of a different flavor on it. If you go to thoughtworks.com and look in their white papers, you can read through this. But these are both really solid, well researched, industry backed AI readiness models to help convince your leadership team if you need that, or just help you do an internal audit of Are we doing the right things to make ourselves ready to reap the benefits of all this experimentation? The last thing is that organizations who are doing really well with AI right now are experimenting by solving real customer problems. Again, space exploration and going to Mars is great, but that is not sustainable for your whole entire organization to be experimenting with going to Mars, it just costs too much money.

It distracts too much from the core business problem. It does not serve your customers. So keep experimentation going. Other experimentation can be really laser focused on real customer problems that you have. And that is how you're going to see the organizational results.

Somewhere, something incredible is waiting to be known. There is so much possibility of how we can build, what we can build, who we can build it for. Right now with AI and agents are just accelerating this. They are expanding our universe. We are definitely in an age of exploration.

The thing I want to urge all of you to take with you into the rest of the sessions today is to find that balance between a sense of wonder and a sense of awe and aiming for Mars and aiming for your moon colony, but also understanding that we need to solve the problems here on Earth and we have to live in this reality. So please, stay grounded, stay skeptical, stay human. Most of all, stay pragmatic. Thank you all.
