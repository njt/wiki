---
url: https://youtu.be/2n41YjR5QfU
date_fetched: 2026-09-22
---

# Software engineering at the tipping point

- **Channel:** Google for Developers
- **URL:** https://youtu.be/2n41YjR5QfU
- **Duration:** 39m 39s
- **Transcribed:** 2026-09-22

---

Hey, everybody. How you doing? Hey. That's pretty good. That's pretty good.

3pm on a Wednesday. That's a Wednesday. Not so bad. Welcome to Software Engineering at the Tipping Point. My name is Adam Bender, and today I'm going to talk to you about something called software ecology.

Ecosystem. A term you might not have heard before. I want to talk about it because it will tell us something about the impact of AI on our developer ecosystems, the developer ecosystems you work in every day. Now, I don't know if you guys have noticed this, but things are getting pretty weird out there, right? Your job in 2026 does not look like what you thought it would in 2020.

I assure you that if you tried to explain to 2020 me what this was going to be, I would not have believed you. It can be a bit overwhelming to try and figure out where all of this change is taking us. Right? Right. And while I can't predict the future, I think that if we study our software ecosystems as they are today, some of the answers we're looking for are a lot closer than we think.

I think that software ecology, this idea, it gives us the perfect lens for framing this particular moment in our industry. Now, I can hear you asking, what is software ecology? And did he just make this up to get on stage? I did not. It is a real term, but before I define it, I want to set some context for you.

So we're going to go a little deep dive on systems thinking. I hope you're okay with that. Hang with me. I promise you it will all make sense when we get there. So let's start with the concept of a system.

Right? A system. It's a group of interrelated elements that act according to a set of rules to form a unified whole. That sounds pretty fancy, but, you know, systems, they look like this. This is a system you're probably pretty grateful for today.

This is what air conditioning is, right? So you have a thermostat that knows what temperature to be. You have an H Vac that's going to jack the temperature up or down. And you have a room, and when the temperature is right, the signal stops. That's a system.

If you're a software engineer, you're probably used to thinking about systems all the time. You design them, you build them, you operate them. But one thing you've probably learned when working with systems is that everything is connected. Everything. That's a theme you're going to hear from me a couple times today.

Now let's move on to an ecosystem which is A particular kind of system, this is a long one, so bear with me. An ecosystem is a dynamic network of interdependent actors that co evolve with their environment, characterized by emergent behavior and decentralized agency. Ecosystems are complex. They're built from complex components. The components themselves are deeply connected.

They have agency, they can make decisions, they can do things. And critically for us, the environment is a part of that system. Right? The environment is a part of the system. You can't separate the two.

So ecosystems are also a kind of complex adaptive system. A complex adaptive system or cas. They're characterized by their ability to grow and change and evolve over time. All the cool systems are complex adaptive systems right? Now, complex adaptive systems, they possess this emergence quality as well.

An emergent property is something that you can't see by looking at any individual piece of the system. You can only see it when the system is all put together, and then you see behavior emerge out of it. And it's that constant change, that learning, plus the emergence, that make it really hard to figure out what's going on in an ecosystem. Now, when you think of ecosystems, or at least when you thought of them before you walked in here, this is probably what you had in mind. And frankly, I'd like to be here right now.

Right. Looks kind of idyllic. But your internal developer environment, it is also a kind of ecosystem. You have all your tools and services, you have complex actors, you have people with opinions and things they want to get done. You have business constraints.

This is a system too. And in fact, it's a particular kind of system. It's a socio technical system. A socio technical system is just kind of a fancy way to say a system made of people and technology. Now, socio technical systems are incredibly complicated.

They're complicated because you start with all that technology and then you mix in the people, right? There's a pretty good chance though, that you've encountered some wisdom about socio technical systems without even realizing it. Have any of you ever heard of Conway's Law? Yes, of course, everyone has heard of Conway's Law. But briefly, Conway's law goes like this.

Organizations build technologies that mirror their internal communication structures. Or informally, if you have a four team group working on a compiler, you're going to get a four pass compiler. That's just how it works. Now, at its core, Conway's law is an observation that the way we build technology is inseparable from the structure of the organizations that build it. The organizations shape what gets built.

Now of course, it's not just the organizational Structure that impacts our developer ecosystems. Right. The values and culture of an organization can have just as profound an impact. Your ecosystem builds what your organization incentivizes. Your engineering culture creates the environment around your developer ecosystem.

See the thing is, once you learn about socio technical systems, you see them everywhere in software development. From your architectures to your postmortem culture to code review to security policy, they're everywhere. The things we build and the way we choose to build them are a reflection of what we value. If we're thoughtful, we can use that knowledge to amplify our values and put them into the things we build, boosting the kinds of outcomes that we want. So now I think I can properly define for you software ecology.

Software ecology is the holistic study of the socio technical ecosystems that produce software. All right, now we're all caught up. It's okay if all that felt a little abstract, by the way, because now we're going to look at a real example. We're going to talk about Google's developer ecosystem. Google has a very large and I would argue somewhat complicated developer ecosystem because over the last 25 years we have evolved truly remarkable capabilities and yes, a little complexity.

Now as an upfront disclaimer, I'm talking about Google partly because I work there, but it is the developer ecosystem I understand the best. My intent here is not to tell you you got to go do what Google did. That's not going to be good for you. Right? You're a different company, you're a different, you're in a different place, you have a different set of trade offs you're worried about.

Now like any engineering organization, Google has made a lot of choices. They were very specific to the needs we had at the time we were building our ecosystem. Your situation different. So a few years ago we wrote a book. Maybe some of you have seen this book before internally, we call it the Flamingo Book.

We wrote this book to give people an understanding of how Google actually worked inside. And we spent half this book talking about things like version control and testing. But the entire first half of this book was spent on engineering culture. And we used to get a lot of questions about that. Why so much an engineering culture?

Well, the reality is if you don't understand the culture that Google has, you can't appreciate why we have the technical choices that we've made. These things are interlinked. Now, in case you didn't read the book, I'm going to give you a brief tour. Don't worry, you don't need to read all this, yes. Take pictures, that'll be fine.

There's a few things about Google that make it somewhat unique. So for example, we're deeply engineering led. Engineers are always in the room when important decisions are being made. We're big on transparency, right? We try to make all the docs and code that we can available as often as possible.

We encourage people to be helpful. In fact, if you talk to anyone who's ever left Google, the helpfulness of their colleagues will be one of the most important things they cite. We treat code review as an opportunity for mentorship and not test grading. We're big on standardization, real big on it. We believe in continuous improvement.

We're big into blameless postmortems. We're big on the idea that sustainability is better than heroics and that automation is better than toil. Now, we don't always achieve all of these ideals, but this is kind of what we're aiming for culturally. And on the technical side, we have our monolithic repository you've probably read about. I'll tell you more about that in a second.

We have trunk based development where every change lands at head every time. When you build a binary at Google, almost every line of code is built from source. That's pretty wild, right? We have a universal build tool chain. Everybody uses it.

We have a global test automation platform. One place runs all the tests, billions of tests a day. We have a global signal for the last green. I can tell you the state of any build looking at one simple internal website. We have a uniform compute environment.

So it works on my machine isn't possible. Mostly we have opinionated developer frameworks and a small set of core languages. And it's this mixture of culture and technology that give rise to what Google is. You can't understand one half without the other. Now, if I had to pick one principle, one principle that seems to have guided us implicitly, if not explicitly at times, it's this thing called shared fate.

Now, shared fate is a term that describes the degree to which an ecosystem and its components are tightly linked to each other. In a high shared fate ecosystem, one component can affect everything else. Right now, in a developer ecosystem, shared fate is as much a technical choice as a social one. You can't just get shared fate by building this, like making everyone use the same technology. You also have to develop social contracts for how you're going to manage that technology.

Now, at Google, our shared fate starts with our monolithic repository. Every line of code at Google, with a couple notable exceptions like Android and Chrome, is in one Shared repository, everything, everything is committed to trunk. There are no branches, there are no versions, everything in one place. This kind of shared fate allows us to apply a security patch in one file and know that within a week every application of the company will have been patched. That's like a superpower, right?

10 lines of code in the right place, in the right place can patch 10 billion lines of application and system software. That's pretty impressive. Right? Now, shared fate isn't always a good thing, right? Shared fate is a choice.

There are places where shared fate might make less sense. For example, in our production environment, we don't want one service to take down all the other services or one cluster to affect an entire region. So we have worked really hard, Google in particular has worked really hard to make sure we don't have the dangerous kinds of shared fate that cause things like cascading failures. Again, the point is shared fate is a trade off. You have to figure out the right place to put it and then make sure that it works well for you.

Now, one of the most interesting emergent properties of our large, of our, excuse me, of our shared fate environment is this notion of large scale changes. Now, long before AI, way before AI, Google's internal tools have made it possible for a single developer to change literally millions of lines of code. Millions of lines of code. Code they will never look at, they will never see again. They might know nothing about it.

Right? We've built tools that make that automatically possible today, and we've been doing it that way for at least the last 15 years. This kind of capability has let us evolve our mono repo over time we can update languages and frameworks. It's really what keeps our internal environment from becoming stagnant. It's no exaggeration to say that without it, we wouldn't be the Google that we are today.

Right? And the folks who work on those tools would tell you it's like it's an unsung hero's kind of job to keep this company moving at the speed it needs to move. The thing about large scale changes is you can't appreciate them unless you understand the cultural components and the technical parts of our ecosystem that make it possible. For example, you need a widespread testing culture. Everybody needs to be writing tests.

For me to be able to do this, we need to have a single platform so I know where to look to get the results of those tests. Right? It's good to have common build tools. So we're all building the same software. I build it, you build it, we get the same outcome.

We need a standard set of libraries, so as we're swapping things out, we're not jumping around trying to find which version of this library is working for you and not me, right? We need standardized code review, we need transparency in the Monorepo itself, so I know which code needs to be changed. LSCs are truly the ultimate manifestation of this automation over toil ethos at Google. And it's only possible because of the entire ecosystem. You can't point at one part of our developer environment and say, that's why LSCs happen.

It's everything linked together. Right now, every developer ecosystem, yours, anyone you've ever worked in, they have these emergent properties. They're usually the things that feel somewhat unique to the place that you work. And that's because they're usually the result of the constellation of choices you have made to shape how you want to work. So every ecosystem has something like this.

Google has large scale changes, you have something else. Google's developer ecosystem embedded into our specific engineering culture has produced a unique set of trade offs. They serve our technical and business goals. However, Google's ecosystem, like every ecosystem, cannot excel at all tasks. We've chosen to optimize for extreme scale security performance.

Right? And we'll do that typically, even if it comes at the expense of developer productivity. Sometimes that's a trade off we're willing to make in our ecosystem. The developer ecosystem of other places like say a five person startup is gonna look different and that's okay. And a five person startup, velocity and agility are like the most important things you can possibly have.

Somewhere in between five person startup and Google is where you are, right? Most of you are inhabiting an ecosystem that's somewhere between five people and 200,000. The trade offs you're gonna make are really important to pay attention to because when you look at the trade offs being made, you can begin to understand the values of the organization. What does it really care about? Not what does it say it cares about, but what does your organization actually care about?

And when you understand those values, you can begin to shape the transformation as it begins to unfold. We're all experiencing a lot of transformation right now, so it would be good to understand where we're going. I hope by now the concept of a software ecosystem is a bit more clear. And I think it's time we talk about the token eating elephant in the room. What does an AI first developer ecosystem actually look like Now?

It would be nice to build one of these out from Scratch a greenfield ecosystem. But I would bet none of you have that luxury, right? You all have to continue shipping software, doing what it is you're doing today, while you're replacing literally every part of it. No big deal, right? Your company is depending on you to continue delivering value and make sure that nothing breaks.

So to make sure nothing breaks, let me ask you a question. How well do you understand your developer ecosystem today? Right? Can you map it all out? Do you know where all the pieces are?

Not just the technical ones, but the social ones? Can you enumerate what your ecosystem is actually built from? How well do others in your organization understand it? What are its strengths or its weaknesses? Where are your bottlenecks?

Where are you constrained or where are you free? What optionality do you have? What kind of emergent properties have you seen in your own ecosystems? And perhaps most importantly, if your ecosystem suddenly had to grow by, I don't know, 10 to 15x in the next 18 months, do you know what would break first? See, the thing about that last question is we're all going to have to answer it real quick because every developer ecosystem on Earth is going through a radical transformation.

Maybe it hasn't gotten to everywhere you are yet, but it's coming. Every single developer ecosystem is going to have to wrestle with this idea of this 10x moment. I wish I could tell you there was a way out of this, but it's coming, right? These are incredible times, but they are also somewhat confusing. All the trade offs we have deliberately evolved over the last 25 years are going to get rebalanced.

You've heard that from probably every speaker here in some form or another. We don't know yet what the future is going to be. We're trying to figure it out. Some folks are predicting productivity. That looks like 10x100x.

We're measuring things in orders of magnitude. That's a lot, right? Suddenly the question of 10x growth is not just a thought exercise. It's like a code red moment for you and your company. You're going to have to figure this out.

If not today, certainly in the next 12 months. Now, before I go much farther, I really want to stress that I understand there's a big difference between generating code 10 times faster and engineering 10 times faster. These are different things. And in fact, at Google, we often say that engineering is programming integrated over time. But the thing is, we're speeding up programming a lot.

We're making the code machine go fast. So we're going to have to figure out how we engineer around that code machine in order to deliver real results to our customers. No one knows yet how far we can push this productivity growth. I really don't know. We could stop next week.

I don't think so. But we're, you know, one thing's for sure, we're going up from here. The problem is, I think we have a lot of work to do because I would bet very good money that what we're doing today doesn't work at 10x. I would bet the way you're building software, the way I'm shipping software, doesn't work at 10 or 100x velocity. Something's going to have to change.

Well, let's start by looking at a standard developer ecosystem. It's simplified. I know. Hopefully you'll recognize some of these terms up here, but let me put them in a form you're more used to seeing them. A complex graph.

Right. In a world with 10x more productivity or 10x more activity, rather, what has to change? Well, let's start with the nodes related to getting code into the code base. Now, notably, I've excluded testing and version control. We'll get to those in a second.

What happens if the green nodes suddenly have to do ten times more stuff? Well, let's start with writing source code. If everyone gets a lot faster at writing code, that's probably gonna be. There's probably gonna be a lot more of it. Right?

That's not good. Now seems like a very good time to remind you of an old Jeff Atwood quote. Software is a liability. Right? So right off the bat, 10x more code, 10x more liability.

We're not into a good place. By the way, you've probably noticed that you can't just hand everyone a bunch of tokens and say good luck. So I hope you're investing in retraining. You are investing in retraining, right? Do you know where all your engineering practices are documented?

Would you know how to evolve them if you had to? Okay, well, something to think about when you get home. Let's move on to the build system really fast. At a minimum, more code is going to mean more compile time. Bigger system, more compile time.

Just what we needed. I'm sure no one at your company has ever complained about slow compiles. Well, guess what? They're going to get slower. And if agents are driving a lot of work, that means more compiles.

So not just bigger, but more. Compilation isn't free in time or compute. And it's possible you've never noticed how much time you spend on compilation, but I can assure you, 10 times larger, you're going to notice something, right? What about the design of all that code? Do you have the right agentic skills to encourage decoupling?

What about the right server frameworks to ensure that you can compose capabilities quickly and safely? Come to think of it, do you know how many ways web applications are served in your company today? Do you know how many different server frameworks are actually running? I don't, actually. How are you going to manage component reuse if agents are writing all of that code?

Maybe you're betting it won't matter. Don't be surprised though. If agents write code that is easy to write and hard for you to maintain. That seems to be the benchmark right now. Agents are good at writing code, but they're not always thinking long term the same way you and I need to.

That code, I'm sure, won't be well factored. That's okay. We'll figure that part out. But the truth of it is, agents are doing a lot of work for us now, and we're gonna have to figure out how we apply that most effectively every single day. At some point, all that agentic work could make your binaries so big you can't compile them anymore.

Or maybe they get so big you can't ship them on phones anymore. Have you ever asked the question, what is the largest binary you can compile? I actually don't know what the answer is, but I know we're bumping into limits at Google. We're getting our binary so big in some places we can't compile them anymore. We'll figure it out, I'm sure of that.

But the truth is, big has lots of consequences. Scale has impacts everywhere. Now, perhaps you're more of a microservices shop and you're looking at me going, why would I ever build a service that big? All my services are tiny. That's cool for you.

Except now I have a question. What's going to happen with 10x more network traffic? 10x more services, 10x more chatter, 10x more network traffic. Are you ready for that? No one gets out of this unscathed.

Scale has effects everywhere. Now, let's say you can reliably compile all this code. I'm sure you can, right? What happens to your code review process? Everyone is worried about code review right now, and I think that's for good reason.

We're putting a lot of pressure on this very human process, right? And in many cases, it is becoming a bottleneck and one thing I know about people, they do not like to be bottlenecks. They act funny when you put pressure on them. Right. With 10x more code, you're gonna get one of two things.

10x larger changes or 10x more of them. So what does that mean for your code reviewers? Well, I'd be willing to bet most of your tech leads cannot sustain the review velocity necessary to see, I don't know, even five of these 10x developers through a day. Right? That's a lot of work.

So what they're going to do in order to not become a bottleneck is they're going to start rearranging their process. They're going to start cutting corners to make sure they don't block anyone, because no one wants to be a blocker. Now, you may be able to solve part of this with AI, I'll give you that. You may be able to deploy AI to make your review process better. Okay?

But that only solves part of your problem. Because if the people on your team aren't writing code, then the only time they're encountering it is during review. But they're not paying as much attention during review. That's what we just said. So who's paying attention to the code base as it evolves?

Well, no one, right? Pretty soon your code base is going to be a mess that no one can understand. Now, in addition to all that source code, we have to think about token management. Because tokens are expensive, as some of you probably know, right? At scale, tokens are a real cost.

We have to factor what happens if everyone in your company starts using 10 times more tokens or 100 times more tokens? Or what if you accidentally spend your monthly budget in a day A. A thing that has happened to a friend of mine, right? If you had to prioritize where you're gonna put that spend, do you know where you'd put it? Do you even have the visibility to know where the tokens are going right now?

See, even in just the first few nodes of this system, this simple system, we're already finding problems. And it is very clear that there's gonna be some challenging second order impacts. Now let's move on to testing and version control. These are a little bit special. Have you ever looked at how much compute your testing infrastructure takes?

I do not know about you, but at Google I never have fast enough tests. I've never said, boy, my tests are going faster than I needed to today. I don't have enough capacity as it is, right? Every change that Lands in version control has to be tested, though. But also, agents love running tests because it tells them whether or not they're doing good work.

So again, the agents are doing additional work and I have more work to do. So with 10 times more commits and agents doing all that work now, how much test compute are you using? Actually, it turns out it could be worse than that, because what we've seen at Google is that as your code base grows, your dependency graph grows quadratically, not linearly, with the size of your code base. So if your code base is 10 times larger and you're trying to test all the dependencies so that you're sure nothing will break, you may have upwards of 100 times as many tests running. Maybe it's 1000 times as many tests running.

That's going to be a really interesting challenge and it's going to be a line item in your budget at some point. That's something you should be thinking about. So now you're not just running tests more often, they may be 100 times bigger. And quite frankly, if they're not going to get that big. If you're not worried about how much test compute you're spending, I'm more worried because that means you probably don't have enough tests to begin with and those agents are yoloing all over your code base with no way to know what is working.

All right, now let's assume you figure that out. You solve compile, you've solved testing. Let's talk about your version control system. Most popular version control systems are not optimized for performance. Not at all.

They're optimized for consistency ordering. Right? That's their job. Their job is to keep a complete record, not to go real fast. What's the performance of your version control system?

How many commits can it take a minute? I assure you it is less than you think. Right. It's not going to scale to 10x the velocity that you need it to. When was the last time you even thought about the performance of your version control system ever?

Not unless you work on git. Right. Honestly, if we're talking about version control performance, something has gone horribly wrong. We are in the bowels of the developer experience here. The bowels.

But that's what happens when you see systemic change. Systemic change finds every corner of your system and says, hey, you paying attention? Because here's something that you weren't expecting. And by the way, for those of you who think you're going to solve this version control problem with a lot of little repos I got some news for you. Talk to anyone who's run that strategy, who has hundreds or thousands of little repos, and I can assure you it's just an entirely new set of challenges.

None of which AI is going to necessarily make easier for you, Right? So far we've hit problems in every node we've looked at, and believe it or not, we've only looked at the relatively easy to find capacity nodes. These are all things you could just find. Like you just take a number, multiply it by 10 and ask will that go good or bad? There's a lot of other unexpected challenges.

So we're going to go through a really quick list of some things I think you want to be worried about. So first, validation beyond just your compute, the validation strategy you use today is probably something like a lot of unit tests and maybe some integration tests. But with 10x more code and 10x more services, integration tests are going to become the most important part of your quality strategy. How many of you are happy with your integration testing setup today? I will note for the live stream there is not a single hand up.

Okay, good. That's what I thought. I am also not happy. Right. Integration testing is really hard and I do not have the tools to do it the way I would like to right now.

Then you have this problem that I refer to as the conjunction of Booleans. In order to ship software today, you request that every test pass, all the Booleans turn green. Everything is good before you ship. That's a reasonable thing to do. What happens when you have a million tests and the actual reliability of the underlying test infrastructure to run a million tests is in question?

It might not be possible to ship software where every Boolean has to be true. So now you're going to need a new strategy, probably something statistical to figure out what are the right tests to run. Because I can't run everything. Okay, what about super extra large changes? It's really exciting that we can refactor and change languages and frameworks everywhere.

Everyone can do it. Do you have the workflows or the social contracts to enable people to manage merge conflicts that are measured in the tens of thousands of lines? Hundreds of thousands of lines. Millions of lines? Probably not.

So we're going to need to figure out how do we build the workflows that support very large change sets moving past each other? If everyone at the company can do it, we we're going to need a new strategy. And by the way, have you ever seen an agentic edit war where one edit an agent makes a change and then another agent comes in and says, no, I don't like that change. Let's do a different change now. It's funny to watch until you realize you're paying for the tokens on both sides of that.

Right, let's move on to release. How many times do you release to your customers today? Daily. If you're doing daily, you're doing pretty good. If you're not, here's a problem for you.

You're gonna get 10x more software, and that software has to land somewhere. And if you're not releasing daily, my guess is that each change is now gonna get a lot bigger. And if there's one thing my SRE friends will tell me, it is that very large changes are very scary. Let's not do that. But the code has to go somewhere.

It needs to get deployed to be valuable. So one thing you're probably going to try to do is release more frequently, and that's good. Our friends at DORA would be proud. They would say, yes, please release more. But at some point you're going to release diminishing returns.

Releasing every second is probably not going to provide a lot of value to you. So somewhere between releasing every second and I'm not quite to releasing a day is the right balance, but still the code will grow. And we're going to need to figure out where do we put that code so that we don't create more risk. How about your internal APIs? We've been talking about building software, but what about the internal developer environment with all the APIs and data?

What I've been telling my friends that I work with is all of your APIs suddenly just became public. They need the same kind of hardening that you would put on anything that you're going to send out to the public Internet. Why? Because agents aren't going to negotiate with you. They're going to find an API, they're going to start calling it, and if they can get access to your data, they're going to do it, I guarantee it.

Right. If an agent can access a data set, it's available to them. So if you haven't put the same thought into your internal data sets and your internal APIs, you're going to run into some very interesting challenges, because agents are going to find things you probably didn't want them to. Well, how about Jevin's Paradox? This is a word we all learned in the last probably 12 months, right?

No one knew what this was or even how to pronounce it, but now we all know about Jevin. Jevin says the cheaper and more efficient a resource gets, the more we use it. And boy, do you see that with tokens. We are putting them everywhere and it's changing how we work and how we think about how we work. We're now optimizing or not optimizing.

We are, we are putting a cost on previously hidden productivity work that was invisible before. So what is that going to do to the way we behave? I don't know yet. Right. By the way, you need to be careful where you put those token engines.

If you put load bearing token engines in place, weird things can happen. Like for example, what if your rollback process depends on an agent to have enough capacity? If you didn't, you know, if someone ran that agent out down the end of its token budget and now you can't roll back, that's probably a bad thing. And speaking of rollbacks, do you guys know why rollbacks work today? Basically, it's because you release software slightly slower than it takes you to defect or to detect a problem in production.

If you can release software really, really fast, faster than you can detect anything is wrong, what does that mean for your rollback posture? Every rollback will now have to contend with multiple conflicting changes landing on top of it. So it's not just enough to release faster. We have to consider the whole system, the rollback as well. That's a really important safety valve.

How about the idea that everyone's a builder? This has been a big conference for celebrating the idea that everyone's a builder. And I think it's cool. Democratizing engineering is cool until you realize you have democratized engineering. You know that tool that you don't like that you wish you could replace at your company.

I don't care what it is, and I won't name names up here, but I'm sure there's a tool you just wish I could vibe code up a replacement for. Okay, multiply that by everyone at your company for every tool they use. What happens to the social fabric of your company when everyone is using completely different tools? Now, if you're lucky, you have a common data substrate. That's a good thing, right?

Then all the data is going into the same place. But what if you don't? Right? Everyone's a builder is cool until you have to maintain all the stuff that everyone built. And let's spare a thought for the technical leadership.

Speedrun. We're going to put all our junior developers through. The reason that it takes so long to become a Lead is because you have to build intuition and judgment and. And expertise to help you make calls. Because when you're leading a team, there's a blast radius that is much larger than when you're just doing it yourself.

When a new grad steps into an environment where they have 50 agents at their disposal, but none of the intuition and none of the judgment, what's going to go wrong? How do I teach 10 years of experience in six months? I don't know yet. And this last one you've heard a lot about, especially in the last talk, if you were here. Human attention is the most precious resource we have.

And right now there's a lot of noise. There's so many agents, so many things demanding our attention. We're gonna have to figure out how to manage this because we don't know how to do it really well right now. We've benefited from the fact that we couldn't make more trouble for ourselves than we could pay attention to. And now that is not the case.

All right, so that seems like a lot. That's because in the system, everything is connected. All the challenges I just mentioned, by the way, you can't resolve any of these by just looking at a single node in the system. Right? You have to look at the whole system in order to adapt to agentic development.

I think we're going to all have to start learning to think in systems all the time. So I'm not going to go through all of these. Take a screenshot of it if you want, but when you're thinking about systems, these are the things you should be worried about. Things getting bigger, the effects over time. Which direction does causality move?

Which nodes are talking to all of their neighbors? What does emergence look like? What are the things that are coming out of nowhere? What about the incentives, both the social and the technical? And that's right, technical systems can have incentives, too.

But. But be careful of the incentives in your ecosystem capacity. I've said it a couple times. I'll hear it at least one more time today. Feedback loops and bottlenecks.

These are the tools of systems analysis. Now, it may seem complicated, but you really only need two questions. Why and what if? Why do we have so few integration tests? What if we had more integration tests than unit tests?

Why do we use these specific programming languages? What if AI writes all the code right? Why is the drill that you are going to use to bore into the heart of your system to figure out how it works? What if will challenge what you find and it will require you to Flex your imagination a little bit. All of you are probably very good at asking why.

I can see it in your faces. You guys know how to ask. Why? Why are we doing this? Why are we doing that?

But what if? What if is more challenging. What if can be scary. If we're asking you to abandon the practices you thought were really well designed for the problems that you had. What.

What if can be scary. What if we didn't test this way? What if we didn't write tests at all? Let's not get carried away, right? But if you allow it to be, what if can also be pretty exciting.

Now, while you're thinking about where these opportunities are, I want to talk to you about a pattern that I've seen. Right? It's this pattern that AI acts as an amplifier. As soon as I learned about this, I started seeing it everywhere. Now, I can't take credit for that idea.

That actually comes from my friends at DORA and their report last year on AI development. They claimed this or they. They found this sort of relationship between or among teams that have really figured stuff out. They figured out how to make AI an amplifier. AI can do more, right?

AI can get you more tests, more documentation, more code, but also more confusion. That's because amplification is a magnitude and not a direction. AI doesn't care where all of that stuff goes. It's just going to give you more of it, right? What DORA really found was that teams that had good fundamentals could apply that amplification in useful directions.

Which begs the question, how are you feeling about your fundamentals? How's the decision making culture in your company? What could you do to improve it? What about your technical strategies? Is anyone looking at developer productivity?

How well do people in your organization collaborate today? What's your security posture look like? How's your code health, your release hygiene, your reliability? AI doesn't solve any of these problems for you. By default, it can amplify the practices you have if they're good.

But if they're not good, it's going to cause more trouble. But even with solid fundamentals, we're in for a real ride. My guess is just this is. This is a guess. You can come check me on it later in 2030.

Our developer ecosystems today are going to kind of feel like 2001 does to us now. And I should point out that in 2001 we were shipping software on CD ROMs, right? That's how far away we might be in 2030. Now, as you continue to build your fundamentals, let me give you some other things you can think about along the way. First and most importantly, you need to know about infrastructure capacity.

You can't deploy the AI and you can't deploy the compute if you don't know how much resource you have to spend. You need a good way to keep track of this. Next, you need validation because you can't, or at least you shouldn't ship software that you haven't validated. But like I've said before, validation is going to change. So you're going to need a new validation strategy.

Now's the time to figure that out. Beyond that, you need isolation because you're going to get a lot of code for a lot of different purposes that previously we didn't use code for. That's okay, but you don't want that cool prototype code to actually find its way into production. So you need to worry about isolation. You need to make sure the fun stuff doesn't impact the money making stuff.

And then finally, you need to worry about abstraction. We build abstractions to keep developers from making bad choices, right? That's why we build libraries and frameworks and whatever. We wouldn't build a web server from scratch today. There's frameworks because there's a lot of ways to get things wrong.

Well, asking agents to make a lot of decisions leads to the same consequences. So we need good abstractions for the agents to hold on to. Don't give them bad choices. Now you're going to have to accept that engineering practices are not sacrosanct. Practices change.

It's the principles that matter. And it's easier said than done. I get that. Especially when some of our principles sort of feel like practices. They feel like they're all the same thing, like testing.

Right. If you've never really thought about why your team tests software the way it does, or why your release process works the way that it does, you're not going to be able to evolve it. Understanding the principles is what's going to give you the power and the confidence to change things. And as we move through this 10x moment, it is a fascinating time to be a software engineer. I'm not gonna lie.

Every dimension of our work is being redefined. We need to flex our creativity more than ever. Right? We need skills to tackle problems like context management, token economics, model drift. We need creativity and we can't get so hung up on the temptation to optimize everything.

We need to encourage exploration. There is a problem that's been keeping me up at night that I know can't be solved. By just optimizing it. And that is, how are we gonna maintain intellectual control over our code bases as we grow? Intellectual control, by the way, is just a fancy way of saying, can humans reason about this thing in front of them?

We've been losing this war for at least the last 15 years. Our largest systems are way bigger than any of us can think about today. That's okay. I think AI offers us an opportunity, right? I think AI might give us the tools to actually begin to, to understand these very large systems as whole systems.

And if by exam, for example, you. Or not example, if by chance, you don't believe me when I tell you that we've been losing this war, let me suggest an exercise to you. When you go back to your team, ask everyone to draw an architecture diagram for your system. See how many different pictures you get, right? We have been losing this war for a long time, so we're going to need help.

The thing is, a lot of our software systems are really brittle. You can break a million line system with one bad line of code or one bad config flag, right? That's the kind of fragile that really makes you think twice before you go about making changes. One potential use of AI that I've been really excited about is this idea that I can get a continuously updated, almost interactive architectural space that I can begin to ask questions of like, what would happen if we took capacity from here and moved it to the east coast? Or what would happen if our user growth suddenly jumped 40%?

Doing that for even a modestly sized system today is functionally impossible. There's just too many variables and too many things you need to know. But AI can make sense of very large sets of data, so I think there's something there. But the thing I like about this problem is that instead of focusing purely on making the code machine go, we're asking how can we deepen our understanding of the things that we have built? That's where I think some of the most exciting problems are now.

There's no denying that change is happening really fast in our industry. And it's happening at a pace probably faster than most of you have ever experienced. One of the most important things all of you can do right now is offer to help someone who is struggling, right? Be a helping hand for someone who has not figured this out yet. We're all moving at different speeds.

We're all dealing with this change in different ways. It is very easy to feel like you're falling behind senior engineers. Be a mentor, Find the people who are stuck and help them. If you have figured out your AI developer workflow, go share it with people. It's not a precious secret.

Help the other people around you. If you're a technical lead, you need to get involved. You need to help steer how software engineering is happening at your company. And critically, if you care about about software quality or software design, you have to use your voice to advocate for it. You in this room are the people who are going to do that.

Your bosses probably aren't right now. At the risk of ending on a somewhat clumsy metaphor, if you imagine our developer ecosystems as living ecosystems, we have all grown accustomed to staring intently at each individual leaf on each individual branch, caring for each tree as though it were some sort of special life form. However, it will not be long before we are all not just managing a tree, but an entire forest. And you can't manage a forest by looking at individual trees. You have to manage a forest by seeing it as an ecosystem.

Right now, here's the trouble with systemic change. It has this quality of happening to everything everywhere, all at once, of being too big for any of us to influence. It can feel right now impossible to grab anything, to steady yourself with the waves of change with washing up on us what seems like weekly. But as we've just discovered in a system, everything is connected. Small actions can have big consequences.

Despite how it might seem, AI transformation is not the sole domain of your company's leaders. They have a role to play. But so do you. As frontline software engineers. In this tipping point moment, you are at the heart of deciding what software engineering is going to be.

From your tools to your workflows, from your engineering practices to your engineering culture. If you can see the systems at work, you can look for leverage. You have more agency than you think you really do. Use that agency to create the future for your organization, for your team, and for you. Thank you.
