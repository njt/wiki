---
url: https://gist.github.com/9972636213519ecf3551adf4aa6d835f
date_fetched: 2026-09-13
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Summary: "Essentials from a Real-World Microservices Journey" — Sander Hoogendoorn, Craft 2025

1. Key Points: The Whole Talk Is a War on Dependencies

Hoogendoorn's thesis, delivered via a tour of his work as CTO of Dutch e-commerce company iBood (13 developers, ~150–160 repos, 40–50 production deploys per day): microservices aren't dead, but most teams do them for the wrong reasons, and the only reason that matters is destroying dependencies.

The wrong-reasons test. "Most people solve the wrong problems with their microservices." Scalability is the canonical wrong reason — "most companies are just too small." Proof: the insurance company's first service handled 68,000 logins per minute on a single JBoss instance. They never had a scalability problem. His closing corollary: if your microservices effort isn't working, "you're probably doing it for the wrong reasons."

Dependencies kill; "technical death" follows. Exhibit A: a public transport company's customer-service dependency graph and an SAP system with 142,000 tables. Exhibit B: iBood's pre-microservices platform, syncing data everywhere via every mechanism, which "totally broke down." Exhibit C: a landscape that took six weeks just to deploy. Companies in this state suffer "technical death" — all time spent keeping things up, no room for innovation, and they die.

Gall's Law beats the big-bang rewrite. The insurance company had 18M lines of COBOL and 12M lines of Java, aging COBOL developers, and six failed rewrite attempts at two years each. "A complex system designed from scratch never works and cannot be patched to make it work. You have to start over with a small working system." Microservices (adopted in 2014, half-understood — "it's new, it's cool") were the mechanism for growing small working systems.

Simple, adaptable, uniform architecture per service. Darwin joke: "the most adaptable architectures survive" — not the 13-layer bank architectures. Every iBood service has the same four layers (domain, repositories, use cases, resources), a pattern he's used in every stack since his Dutch UML book 25 years ago. With 13 people and 160 repos, anyone must be able to open any repo and know where everything is. Hence also the single-stack decision: everything in TypeScript, mobile moving from Flutter to React Native.

Decompose with DDD. Bounded contexts (Product has two meanings — replenishment vs. ordering — so split them), ubiquitous language (one meaning per context), and aggregates as the unit that becomes a service: a boundary, a root entity addressed by id (which maps neatly onto REST resources), and outside references only to the root. Boundaries are not static — iBood's differ from two years ago. Cohesion is about code that changes together, not technical layers: "don't put all your repositories in one layer."

Break up the front end too. The common pattern — services below, one app on top — leaves the app holding all the business logic and dependencies. iBood split the UI along the same domain seams (products, basket, checkout, account, orders). Apps talk only to services; no app touches a database. Deploy "my iBood" pages without touching checkout.

Small releases are the point. "Missing the bus is not a big deal if you know there'll be another one along in five minutes." A bad release costs five minutes, not a quarter. Small deltas also make testing easier. The opposite spiral — distrusting releases, making them bigger, releasing once a year — ends in bankruptcy (his anecdote).

Automate everything; build quality in. Deming: build quality in rather than inspect it in. Full pipeline from check-in to production in ~30 minutes, with pipeline definitions as shared code. Trunk-based development, no pull requests, no code reviews — in his context. He concedes regulated organizations with 30 teams on one mobile app may need PRs.

Services own their data. Sharing a database just relocates the dependency graph. Snapshots freeze state where history matters (an order stores its product data at purchase time — a later price change doesn't mutate the order). Polyglot persistence: document DBs by default because an aggregate persists in one statement.

Ways of working must change with the architecture. No Scrum (quit 15 years ago), no sprints, no retrospectives, no Scrum master, everyone is their own product owner, no estimates beyond ballpark, fewer standups, collective code ownership. Event storming, pair/mob programming, and self-organizing "micro teams" — because "no two items that come off a board actually require the same set of people."

Simplify to amplify. Dee Hock's principle is the talk's second memorable slide: simple purpose and principles produce intelligent behavior; complex rules produce stupid behavior. Amsterdam removed traffic lights at a crossroads and drivers started making eye contact and negotiating. Remove rules and people communicate.

2. Pithy and Provocative Quotes

"Most people solve the wrong problems with their microservices." — Which, he notes as a former consultant, is "nice... you can get hired."

"They suffer from what I call technical death, meaning they spend so much time keeping everything up that they have no room left for innovation anymore and they just get killed. Companies disappear because of that."

"A complex system designed from scratch never works and cannot be patched to make it work... you have to start over with a small working system, there's no way around." — Gall's Law, backed by the insurance company's six failed two-year rewrites.

"It's not the strict ones, the ones with 13 layers in it that normal banks would have. It's the ones that you can adapt to." — his Darwin bit: "the famous Java developer. He has a beard. Charles Darwin."

"Missing the bus is not a big deal if you know there'll be another one along in five minutes." — "This is the slide to remember from this talk... I'm going to ask you tomorrow." (He credits "Jason.")

"If it hurts, do it more frequently and bring the pain forward. This is still about software development, right? No other domains are included here." — quoting the Continuous Delivery author.

"It's the first rule of microservices Club. You don't talk to others." — the Fight Club joke about services reaching into each other's databases.

"Simple, clear purpose and principles give rise to complex and intelligent behavior... complex rules and regulations give rise to simple and stupid behavior... The trick is to simplify, to amplify." — Dee Hock, "the second thing to remember from this talk."

"I stopped using scrum 15 years ago. I don't like it very much. But that doesn't mean we do Kanban... There's like 50 shades of ways of working between."

"My youngest son once asked me, 'Dad, if you were not a programmer, what would you do?'... 'I would learn to write code as fast as I could.'"

"We're now solving everything with AI and microservices are passé." — opening snark at the "really full room" for a microservices talk.

3. Tools, Practices, and Methodologies

Martin Fowler's microservices definition — the short definition (small services, own processes, lightweight mechanisms, built around business capabilities, independently deployable via fully automated deployment). Use it as a checklist/table of contents for everything you must solve.

Gall's Law — complex systems designed from scratch never work. Use it to kill big-bang rewrite proposals; start from small working systems and grow.

Four-layer service architecture (domain / repositories / use cases / resources) — his standard micro-architecture. Use cases double as the authorization unit: only people with that use case in their scopes may execute it. Repositories absorb data from anywhere — database, another service, external CRM — and the service owns it. Apply identically in every repo so any developer can navigate any service.

Single-stack policy — all code in one stack (TypeScript, React Native replacing Flutter). For a small team, multi-stack knowledge is "unmaintainable."

DDD bounded contexts + ubiquitous language — split the domain where one concept (Product) plays multiple roles and has multiple meanings; each context gets a single language.

Aggregate pattern (Evans) — objects changed as one unit, demarcated by a boundary, with a root entity addressed by id; outsiders reference only the root. "You turn the aggregates into services."

Micro frontends per business capability — split the UI along domain seams; apps talk only to services, never databases; independent repos and pipelines. Not 1:1 with services — most apps talk to several.

Continuous delivery with tiny deltas — deploy small, deploy often; small deltas make testing easier and bad releases cheap.

Trunk-based development — push to trunk ASAP to avoid merge conflicts ("I'm too old for this shit" re: merge tools); it pushes you toward ever-smaller changes. Replaces PRs and code reviews in his context.

Full CI pipeline as code — format/lint/prettier → unit tests → Sonar → containerize → acceptance → load tests → production, ~30 minutes. Pipeline definitions live in a shared script, so a change propagates to all ~150 pipelines.

Michael Feathers' unit test rules — not a unit test if it talks to a database, communicates across the network, needs configuration, or can't run in parallel. Every test sets up and tears down its own state.

SonarQube, coverage on new code — free, and the trick is measuring coverage on new code, not just the base; 80% threshold. "Not because we like it... it's a way to keep the discipline going."

Services own their data + snapshots — no shared databases; every domain situation needs a decision about where data lives; orders store a snapshot of everything at purchase time.

Polyglot persistence — document DBs (Mongo) as default because an aggregate stores/loads in one statement; Firebase for auth; relational only for hierarchical data.

Event storming — to figure out how processes work and how the domain is structured; "fits really well into DDD and microservices."

Pair / mob (ensemble) programming, dynamic huddles — ad hoc, remote included; a serious breakdown two days ago drew the whole team in without anyone asking.

Micro teams / self-selection from boards — three boards; people pick items they like, are good at, or want to learn; items range from half an hour to a couple of days.

No-ceremony ways of working — no sprints, retrospectives, Scrum master, or dedicated product owners (everyone is their own); no estimates beyond high-level ballpark; experimenting with fewer standups.

"Simplify to amplify" (Dee Hock) — remove rules and process so people communicate and behave intelligently; experiment by removing things, and "if it doesn't work, put it back in."

His open-source TypeScript microservices framework (on GitHub) — contains the architecture documentation and code examples.

Vibe coding — mentioned with a wink: "you need to do vibe coding, otherwise you're not hip these days."

4. Unanswered Questions and Omissions

The monolith rebuttal never arrives. He mocks "microservices are dead, long live the monolith," answers "yes and no, probably," and moves on. He even concedes the cohesion principle works in a modular monolith ("you should, by the way"). The talk never explains why iBood couldn't be a well-modularized monolith with the same CI/CD discipline — and a 13-person team with 160 tiny repos arguably makes the opposite case. This is the elephant in the room, acknowledged then dodged.

Observability is completely absent. The "next bus in five minutes" strategy depends on knowing you missed the bus. No monitoring, alerting, tracing, or logging across 150+ services. He mentions "a serious breakdown two days ago" but never how it was detected or diagnosed. How do they know a release is bad before the next one ships?

Runtime coordination across services. How do business processes spanning multiple services actually execute? No sagas, orchestration, choreography, or eventing — everything appears synchronous HTTP. iBood's original disease was data synced everywhere; the talk says microservices "worked" but never explains how cross-service data flows work now.

Consistency beyond snapshots. Snapshots solve order history, but what about data that must be current across contexts — e.g., stock levels shared between replenishment and ordering? Eventual consistency is never discussed.

Versioning and compatibility. 40–50 independent deploys a day, and no word on breaking API changes, consumers on old versions, or contract testing between services.

Scaling the team. Everything is premised on 13 people with collective code ownership and no reviews. What happens at 30 people? With juniors? How do you onboard into 160 repos with no code reviews and no documentation beyond the architecture?

Quality control without reviews. PRs are dismissed because feedback is "usually on the linting or the formatting" — but that indicts how reviews were done, not reviews per se. What catches design flaws and spreads knowledge besides ad hoc pairing is left unanswered.

Cost. 160 repos, ~150 pipelines, containers, multiple data stores, six countries — the cloud bill and operational overhead never appear.

Security. Use-case scopes are the only security mentioned. Nothing on service-to-service auth, secrets, or payment/PCI concerns for an e-commerce company.

The insurance company epilogue. "Sort of worked" is the last we hear. Did the 18M lines of COBOL ever die, or did microservices just grow around the mainframe on the fourth floor?

AI. He opens with "we're now solving everything with AI," drops it, and returns only as a vibe-coding punchline. What AI-assisted development does to a no-review, trunk-based, 13-person operation is never asked.

The cost of moving boundaries. He says service boundaries differ from two years ago and change constantly — but re-splitting services means data migration, and that cost is never addressed.

I got word from my people that we can finally welcome to the stage Sander Hogendorn. I'm ripping off his name. But anyway, he's coming here to talk about architecture that works. So without further ado, let's give a hand to Sander. So, good afternoon.

It's already afternoon. Right, Sorry we're a little bit late. We had some technical issues. Usually my laptop just connects and it works. It's a MacBook, right?

It should. And now it's over there and I have to read my slides from there. So. Going to give some strange effects, probably. So it's a really full room.

I didn't expect that for a microservices talk. Right, because that's old, right? We're now solving everything with AI and microservices are passe, right? As we say in Dutch, French, whatever, right? So.

But I've been doing this shit for quite this stuff for quite some time. And so I thought it would be a nice idea to just show you a really quick overview of a lot of the stuff that you can take into account. Well, might even have to take into account. If you go into a microservices world. It's going to be high paced, especially since we're late already and I'm keeping you from lunch and presumably beer as well.

So we're going to talk about microservices a little bit, but I only have like 45 minutes, right, so how much can you say about microservices in 45 minutes? Well, you can only say enough if you talk really, really fast. Now, luckily, that's one of my core things I can do. So that's what we're gonna do. So, first of all, right, microservices is over the hill, right?

The hype is gone. And people say, yeah, well, AWS threw it under a bus and they went on to do something else. Yeah, that's part of aws, right? And then people say, well, microservices are dead. Long live the monolith.

So we're going back to building monolithical systems. Yay. Good idea. The death of microservices. So everybody writes about, are microservices dead?

And well, yes and no, probably. Right? Because the question is, are we doing microservices? Are we doing something else? Do we?

The right way? Is it difficult? Yes, it is, etc. Etc. So let's start with Martin Fowler's definition, right?

And this is as Martin Fowler writes his short definition. Not sure if there's a longer one, but he says a lot of things in this definition, I'm going to use the definition just as the table of contents for this talk. So he says, well, it's an architectural style and it's an approach to developing a single application. Not sure what a single application is though, but as a suite of small services, all running in their own processes and communicating with lightweight mechanisms, which makes it easy to communicate with other things that also using the same protocols. Right.

And they're built about business capabilities, which is the crucial part about this thing. And they run an independently deployable machinery that is fully automated in their deployment. Right. So as an example, my current company I work for, I'll explain a little bit about that. We deploy about 40 to 50 times per day to production.

Right? That's for some people, that's slow, for some people that's fast. Doesn't matter. It's all relative, Right. For us it's enough sort of.

Not always, but it was today, so. And they can be written as a result of you running in losing really lightweight protocols. You can write your stuff in any programming language you like as long as IT support HTTPs. And you can use different data storage technology. So this is basically the stuff you need to take into account.

So to summarize it all up, so you need to first identify what are the problems that you can actually solve using microservices. Because all a lot of people choose microservice for the wrong reasons, which is nice as a consultant. Right. So you can get hired and you can solve those problems in different ways usually. But that's one thing they need to talk a little bit about architecture.

I'm going to keep that part really, really simple actually. Just show you what we do. We also think about as the application is not such as one application, but as a platform or a collection of small applications that actually inhibit the same characteristics as our services do. And then you have to think, well, my clicker is too far out of the reach of my laptop, I suppose. Then you need to think about how to break it down.

So what are the patterns that you use to break down. Well, your. Well your business domain into reasonable chunks. And after that you need to talk about CICD, of course. Right.

That's why we can do 40 to 50 times deployments per day or even more or less, depends on the day. And a little bit about testing. We all like testing, right? Yeah. Okay.

This was the show of hands of people who like testing. There's nobody like we have to do it anyway. Especially if you do distributed landscapes. You have to be very aware of Your testing and then we talk about data. And the last part is a bit about the ways of working.

Do they change when you do this stuff? In my opinion, yes, they do. So this is the table of contents. Right. So let's start with problems.

First of all, I'd like you to to understand that I always adhere to this principle, saying what works for us might not work for you. Right. So take from it what you like, the stuff you don't like. Just leave it here and have some more beers and think about it later today. Okay.

Again, still 45 minutes. Well, 40 left, more or less, but. So the question is, what do we do? So now I'm going to introduce myself. This is me.

This is a really big picture of me, actually. I'm a single independent dad. I speak a bit, I write a bit, and most of my time I spent writing code. It's what I love to do the most. I've been doing that for 47 years.

Yes. And I still love doing it every day. I haven't written any code today, but I probably will. So I'm currently the CTO from a company called Iboot, which is an E commerce company based in the Netherlands. And we're active in like six different countries in Europe.

Not here, unfortunately, but that's it. If you want to connect to me on LinkedIn, feel free to do so. That's it. Right. So just to show you that I'm a programmer, this is me.

I'm double object oriented. You can't make this up, right? It's true. It gets worse, by the way, because I live in Amsterdam and this is the street I live in. This is actually taken from my window.

So that's a true story. So I do contribute a little bit to open source. This is our framework for doing microservices in typescript. It's nice and there's a lot of nice code examples in it. So.

Yeah. So what problems do we solve with it? Well, to be quite honest, I think most people solve the wrong problems with their microservices. I'll show you some examples of companies that I worked for that tried to do microservices to solve their problems. By coincidence, these two examples sort of worked.

So first of all, I started working for this insurance company. I would never recommend anybody to work there. It's a terrible company. And the problem they had is they had a lot of legacy. So they had an on premise mainframe and this being an insurance company in the Netherlands, where the building is below sea level, the mainframe was on the fourth floor instead of in the basement.

Differences. And so they had a lot of code on that. They ran about 18 million lines of COBOL and 12 million lines of Java. You could reason about what's worse, but this is the cobol. It's in Dutch, which makes it worse.

It's also very much abbreviated. So it gets worse. Unmaintainable stuff. Right? Now the problem they had is that most of the COBOL developers that were working for the company, they were actually sort of aging.

So at some point they wanted to retire, right? And that means that they had to get rid of all this code and they couldn't. It was way too complex, right? And here's the thing, here's the thing called Gold's Law. And John Gold basically says a complex system designed from scratch never works and cannot be patched to make it work.

This they tried six times, spending two years in every attempt to try to write the next big system. It doesn't work, right? The thing is you have to start over with a small working system, there's no way around. You cannot just write a very complex big system and think that it will work in production. It just doesn't.

They tried six times. It's living proof. And then I came in, I said, oh, we need to do microservices. I was probably talking to Sam just before that happened, it was in 2014 or something. And then I said, yeah, yeah, we need to do microservices.

And I said, what is it? And I'm like, how do I explain this? It's new, it's cool. It was the hype at that point in time. And they said, well, let's do that.

And then we went. We didn't know what to do, by the way. So this is the management team of my current client. They said, can we be in your slide deck? I said, sure.

So I took the picture and it's in my slide deck now. Nice people. Unless everything breaks down and they get really nasty like yesterday. And it's an E commerce company. So they had a platform, it started with one deal on a page and then it grew and grew and grew and grew until they didn't have like 130 million in revenue.

So it started easy and then they added all sorts of stuff to it, like stuff written in php, open source systems that they manipulated or destroyed basically that could not upgrade to the next version. So we're running 10 versions behind on the ERP system, stuff like that, right? Stuff that happens in everyday companies. And then they started doing microservices. With really weird names.

So it didn't work because of the weird names, of course. And they ran on every cloud you could think of connecting to every part of infrastructure that you can find, and every marketing tool, every payment provider, whatever, right? And then they realized that they had, like, data sitting everywhere. I need to click here, probably. Yeah, that's close enough.

I'm gonna stand here. So they needed to. To have the data everywhere. So they tried syncing the data everywhere using every mechanism that you guys know, right? Think of a mechanism.

We have it. It's in place, and it totally broke down about five years ago. And they're like, can you solve this? I said, yeah, microservices. It worked, by the way, and it wasn't interesting.

The problem here is that with every system is that dependencies will kill you anytime, right? And this, by the way, is a. A graph from a public transport company in the Netherlands. And this is. Every rectangle is a system in the landscape.

This is just their customer service department, by the way. This is an SAP system. It has 142,000 tables. That's why SAP consultants are very expensive, right? If you want to rig meal money, then you become an SAP consultant.

You learn all the tables by heart. So this is the problem that a lot of companies have. They suffer from what I call technical death, meaning they spend so much time keeping everything up that they have no room left for innovation anymore and they just get killed. Companies disappear because of that. And that's really sad, actually.

But the good thing is still a lot of people go into microservices for the wrong reasons, right? Everyone says, yeah, it's for scalability. Most companies do not. Unless you're Netflix, of course. But most companies do not really have scalability issues, right?

Most companies are just too small. The insurance company. The first service we built was the account service. We tried logging in. We could log in on a single JBoss instance 68,000 times per minute.

And we're like, I think we just solved our scalability issues. We don't have any. The company didn't expect 68,000 logins in a minute. A lot of companies don't do that, right? So don't go here for the wrong reasons.

So what's next? Architecture. You can say a lot about architecture. I'm going to just say the microarchitecture of a single service. And so we investigated that for a long time, and then I realized at some point in time that, well, if you have a very simple architecture, just like the famous Java developer He has a beard.

Charles Darwin says it's the most adaptable architecture that survive, basically, right. It's not the strict ones, the ones with 13 layers in it that normal banks would have. It's the ones that you can adapt to. And I realized that long time ago. I wrote this book, which is on the next slide.

It's in Dutch, so nobody knows how to read it. It's a weird language. It's not as weird as Hungarian, but it's. It's weird anyway. Well, yeah, maybe it's weird.

Would it be weirder? It is weirder. It's confirmed. Dutch is weirder. Anyway, at one point in time, I wrote this book about UML, like 25 years ago.

And there was this simple architecture in it. And I realized when we started doing microservices, that was still a very nice architecture. And it's extremely simple, right? So I'm not saying adopt this architecture you can, but the goal here is to say, if you do microservices, make sure that all these servers have that a really, really simple, adaptable architecture. That's the core of it, right?

Ours looks like this. It always starts with the domain. This is the part of the domain that this service reasons about, right? And then it has repositories, basically pattern that helps you get instances from wherever they come from and put them into your component and make sure that you can reason about it and you can validate them, et cetera, et cetera. So the repositories are there and then we have use cases around and you're like, what use cases?

They're from previous century. Could be. But we use them basically as well, the container for process logic. We also use them for authorization. Right?

If there's a use case that is called whatever Manage profile, then only people who have that particular use case in their scopes, they're allowed to execute that use case. So it helps us make the authorization model easy. Actually. On top of that, there's the resource classes. This is how you get to the service.

This is where the calls come in, the requests come in, and they were sent through the other layers. And then you need to figure out where your data comes from, right? It could be coming from a. A database, quite normal. Could be coming from another service, it could be coming from an external party, from your CRM system, wherever it comes from, right?

That's one of the nice things about microservices is that your data can come from anywhere and you just absorb it and the service owns the data. So that's one thing Right. And the nice thing is I was looking back into code bases and actually wanted to show code, but I didn't have to run off and on the stage again. So that I'll probably do it like this a bit. I realized that most of my code bases, ever since I wrote the book, basically adhere to the same architecture.

This is a. NET code base from 2014. It has the same layers in it, actually, if you look into the code, this was code from that project written in C. This pretty much is a repository. It's the same pattern I still use.

And it's the pattern that comes from the DDD book, by the way. And it's a nice pattern. You can write it in any language. This is in Java, right? This is, by the way, functional style of writing Java.

But I'm not going to talk about monads and functional styles today. This is one in Python. It's ugly. No, I don't. That's not that ugly.

But it has nice orange colors in it, right? So it's cool. This is one in TypeScript. It's a bit shorter and they all do the same thing, they have the same responsibilities. And you can use that over and over again in any language, in any platform.

Right. It's a simple way of doing stuff for us. That's essential. My team is not that big, right? I don't have 1500 developers in my team.

I wouldn't want 1500 developers in my team. We're a team of 13 people, including myself, and that's it. So I have to deal with that as a result. One of the choices we made at the E commerce company that I work for is we put everything on a single stack with 13 developers. You cannot put your code in different stacks because.

Because you don't want to have to have knowledge about all these stacks. I kind of write part of it in Java, part of it in TypeScript, part of it in React. Well, it's also TypeScript, but part of it in Python. It's unmaintainable. We couldn't do that.

So we're moving everything to a single stack, including the mobile app, which is now moving from Flutter to React Native, also typescript, and then everything has the same architecture. For us, that's a necessity. We about 150 to 160 different repos. They're all quite small, but I need to be able to open up any of them and understand what the code is built like, I need to see where the layers are, where the classes are, where the different responsibilities, the different patterns are. So that is basically how we roll.

We have everything in the same architecture. I can now, if I could reach my laptop, which is normally here, but sorry, open up any of our repositories and understand what's in there for us, that's essential because we are a small team, right? So that's a choice we made, by the way, a little bit of documentation on the architecture is on the website on GitHub, actually. So what's next? Well, we also decided that we're not building a single application.

We decided to build a collection of small applications. So the nice thing about our services that are independently deployable, right, that means that I make a change to one of our services and I can deploy it to production. And if all my tests are green and everything works, then it's just there and I don't have to check and validate and test the rest of the landscape. And we wanted to do the same for our user interfaces, we broke them up as well. And again, that's all about breaking down the dependencies, right?

A lot of people who are doing microservices, they do this basically. They build a bunch of services that own the data and do stuff, and then they build an application on top of it that talks to the services. This is quite a regular architecture, actually. The problem here is that you still have a lot of dependencies, right? You broke down the dependencies in the services.

I'm going to go back one. But the application is still holding everything. It has all the business logic. So we decided to break that down too. And we did it actually by looking into the business processes that we are implementing.

And the question is, are there parts of the domain that are owned by specific parts of the ui? So we're an ecommerce company. So I used this example, actually, I had this example way, way before in my slides, even before I went to do E commerce. And actually, E commerce is. It's a bit more complicated than I thought it would be from reading all the literature that we have in this industry, right?

So you select a product, you select another one, you put it in your basket or in your cart, we call it a basket, actually. And you go to checkout, you register yourself, you register payment, and then you just show your order. This is basic, basic process, right? Now, if you look at this, you can see that there is actually different parts of the domain that are sort of like owned by different parts of the process. So if you would break these up into, let's say, smaller applications, you could also reason about that.

This one is about products. That one is about the basket or the cart, about your accounts and about orders. We did that. We broke them up into different parts as well. Also revolving around a small part of the business domain.

By the way, there's not a one to one from an app to a service. Most apps talk to different services and they comprise their own part of the domain that is useful for that part of the ui. Now, the funny thing is, if I would write some code into the orders app, meaning probably show some additional information about people's orders or doing whatever is necessary, I can deploy that part of it without having to touch the other part. And that is a big thing. Right.

If I can update my, we call it the My iboot pages without having to touch the checkout. For us, that's pretty useful because we don't like touching the checkout. It works, it's great. But I don't want to touch it if I don't have to. Right.

So we're using the same things on our apps. Does that work? It works quite nicely, actually. We have independent pipelines for them as well. We have independent repos.

And the good thing is the apps only talk to the services and the services talk to the data. None of the apps talk to a database. That's an architectural choice we made and it works quite well for us. The next question is, how do you break down your domain? Now this is where we have to sort of go into, let's say domain driven design.

But actually the funny thing is that if you look into the history of computing it. Actually this is my team, by the way. It has been the same one cannot use my click arrow from here. Yeah, so it has been quite similar all along. Right.

So in the 70s we already talked about low coupling and high condition. So you put the stuff together that belongs together, not because they're all repositories, but because they're all reasoning about orders or products or accounts or profiles or whatever. Right. So you put the stuff together that actually belongs to the meta and sort of like changes together. And it's not the data, it's the code that changes together.

Right. So if you can do this, you're already in a much better place. You can do this in a modular monolith as well you should, by the way, if you do monoliths, and that's one thing. And if you look down into literature, the UNIX philosophy says more or less the same thing. You write programs that do one thing and do that one thing really well.

And if you look into Bob Martin's work, he has the single responsibility defined, right? And it says, gather together the things that change for the same reasons. This is the thing, right? The things that change for the same reasons. That is what you want to do.

So that doesn't mean put all your gateways in one layer. Put all your repositories in one layer because they don't change for the same reasons. Your order gateway and your order repo change. If there's something with your orders, right? Your product gateway, your product repo and the use cases on top of it, they change.

If you do something about your products that is the breakup that you want to do, it's pretty simple. Well, it's not that simple, but it's, it's straightforward and the patterns all point to the same direction, right? And this thing, well, it's a Python file, but it doesn't really matter. This is already like for me, this is a code smell, right? A file that has 352 lines, that's big.

In my book, I write very small functions, usually one liners or one statements. So this is the thing you do. So where do you go? So you go look into the Domain Driven Design book. I assume you've all read the Blue Book.

Yeah, well, yes, the Eric Evan's book, of course. But the problem with the book is it's excellent. It's also really hard to read, actually. So it's one of those books that if I can't sleep, I put on my nightstand and I'll open it up, I read three pages and I fall asleep like an angel. Right?

But it's the quintessential book for this industry, right? So here it goes. So Eric Evans says when you model larger domains, it becomes progressively harder to create the single unified model. It's what we all try to do, right? We create this big data model of everything we have in the company.

And the more you extend it, the more you change it, the harder it becomes to maintain it. Kind of makes sense, right? This is still about dependencies. And then he says instead of creating a single unified model, you create several. And they're all valid within their own bounded context.

And you're like, what is a bounded context? If you haven't read the book, if you haven't seen anything about the way in ribbon design, you, you might think, what is a bounded context? Well, let me give you a very straightforward example. This is the very complex domain model of an E commerce company. Again, I had this in my slides way before I started building this stuff.

And it's way More complicated. But this is a good example, right? So this part of the model deals with replenishment, right? If you're out of stock, you want to reorder it with whatever vendor has it in stock for the lowest price. That's part of what you do.

The other side of it, if people order the products, well, there's the orders and payments and there's the customers and stuff. That's another part of your E commerce domain, right? They are both important and they both reason about products. The only problem with the product is that it plays two different roles. It plays a role in the replenishment and it plays a role in ordering stuff.

Now, you could think, like, what? What does that mean? Well, that means that it has behavior and probably properties that belong on that side, and it has behavior and properties that belong on this side. So it actually has two different meanings. That's basically the next pattern in the book.

It's called the ubiquitous language. Simply translate it. Everything without in a single bounded context adheres to a single ubiquitous language meaning. Everything has a single meaning. Product doesn't have that.

In this model, product has different meanings. So you might say, what if I split this up, right? What if I do this? What if I create a thing, whatever it is, that does everything with ordering and another thing that does everything with replenishment? Then I have two different things, and they're only really thinly connected by this really thin line between the two different products that allows you to do that.

So we started doing this in 2014, 2015. It's like, oh, this is the pattern we use to create microservices. And after a while, we realized that they were still quite big. And then we were looking down into the book again, and then we came into the next pattern, which is the aggregate pattern. There's a lot of discussion about the aggregate pattern.

I like it in the way it was originally written, to be honest. So here's the definition. Basically, Evan says aggregate is a group of associated objects which are considered as one unit with regard to data changes. Again, that's the thing, right, where everything changes for the same reason. Well, you save it for the same reason as well.

And then he says the aggregate is demarcated by a boundary which separates the objects inside from those outside. This is typically what a microservice is, right? It's a part of your domain that is sort of separated from the other parts of your domain by the boundary that is the microservice. So it's a really nice way of looking into microservices. The next part of it is as well.

Every aggregate has a root. This pretty much aligns very well with how you look at resources. Right? You have a URL that goes into last year in the same tent, it rained while I was doing my talk, so I had to shout basically to get across. And all the field was flooded, actually, so this weather's already much better.

Each aggregate has a root. The root is an entity, and it's the only object accessible from the outside, usually by its id. Right? So you go to say, I'm going into the product service and I'm going to get this product by its id. That is the address, basically the thing lives in.

So it makes perfectly sense to do this mapping. And then you get to the point that you say, well, one aggregate might be a very interesting pattern to start doing microservices. He says the root can hold references to any of the aggregate objects, but an outside object can hold reference only to the root object. So you cannot go pass the root object into the sub objects that are inside of this aggregate. This is exactly how we use microservices.

So for us, this was the next step, well down the path of getting to the right level of aggregation. Basically, it works for us, right? Doesn't mean it's going to work for you. So this is basically it. You split everything up into bounded context, you define the aggregates, and basically you turn the aggregates into services.

Sounds really simple. It's not, right? This takes a lot of negotiation, figuring out when stuff changes. And it's not static. It means this is going to evolve over time.

If I look into the boundaries of our services that we have right now, they're different from the ones we had two years ago. And it changes all the time. You need to be able to adapt to those changes. We're in the forest, not in the desert. If I listen to the brilliant talk, I come back.

So that's it, right? So that's how we break it up. Next step is if you want to have fully automated deployment machinery, as the definition says, and you want that to take away basically the pain of having to put stuff to production, you go into continuous delivery. So the question then is, is microservices a good architectural style to do continuous delivery? Or maybe the other way around?

And yes, it is, because if you try to deploy and put into production stuff that is really, really small, it's much easier than if you have to put really big chunks and big applications and landscapes as a whole, with 40 different systems in one go into production. The chart that I showed you on the dependencies, it took them six weeks to deploy the whole landscape. Six weeks? Just the deployment, Right. There was no coding, no testing whatsoever.

Just the deployment was six weeks. That's terrible, right? That means you're going to lose speed, right? And if speed is relevant to you as a company, then you go into smaller parts. It's just that simple.

So, yeah, you can go in to deliver small parts. And then just the author of the continuous delivery book, he says, if it hurts, do it more frequently and bring the pain forward. This is still about software development, right? No other domains are included here. So if you have this really big system or this big landscape that you need to redeploy, you wait.

And the longer you wait, the bigger the set of changes becomes that you need to deploy and that you need to test, and you do integration tests and system tests and everything that you need to test before you can deploy the next version. That takes time. And what you see with organizations who do this is that they get slower and slower because they don't trust that. That chain. They're in the desert, basically.

So they don't trust that change set. So they make it bigger and bigger. Oh, yeah, this needs to be there too, because we can only release once every three months and then once every four months. I worked for a company that could only release once a year, right. Imagine that they went bankrupt, by the way.

Of course. But that's what happens if you do this, because again, dependencies between the different systems, your services are going to kill you. The thing to. If there's one thing to take away from this talk, it's this thing. This is my favorite quote from the talk.

It's not my own, it's Jason's. And Jason says, so keep this in mind wherever you go. This is the slide to remember from this talk. I'm going to ask you tomorrow, if you talk to me in the hallway and say, what do you need to remember? And you say, yeah, something with a bus.

Here's the thing. Missing the bus is not a big deal. If you know there'll be another one along in five minutes later, then you're okay. So last night when we got back from dinner, we went to the Metro, and then they said, the last train is coming, and we're like, oh, we better make that train, because otherwise we'll have to wait here until, well, five in the morning or whatever. The first train comes along, then you're in trouble.

But if the next train comes in five minutes, I don't care if I miss it, Right? It's not a problem. And this is how we started to write code. Microservices help you to get to this point where the next bus does come along in five minutes, we deploy stuff to production where people say, isn't that a bit early? And we're like, doesn't matter.

Because if there's something wrong with it, there could be. Right? Then we'll just release again in five minutes time. And that's fine. Not for every industry, by the way.

Again, this goes for us, right? So for us in E commerce, this is the strategy go. Because marketing always wants to go faster. So we help them to go faster by doing this. And that means that if you deploy different parts of your landscape in very small parts continuously, you can go much faster.

And the nice thing is the deltas between one release and the next are so small that testing becomes easier actually, of the whole landscape. And that is a good thing, right? So that's where you go. So yes, microservices help you to get rapid feedback, and rapid feedback helps you to better or become more productive, as the desert people would say. I'm going to use this forest and desert metaphor a lot.

I think so, yeah. It doesn't matter which stack. You can do this on any stack. The thing you have to do, though, is you have to automate everything. Everything you can think of to automate.

Automate it. I'm in a lucky position that I have people on my team. Where is. Oh, here, this guy. He is incredible, right?

He's our ops engineer or whatever you call it. And he's incredible. He can write pipelines and automated stuff faster than anybody else I know can do. So I'm really lucky to have him on my team. And we automated everything.

And that sort of adheres to what Edwards Deming says. You always have to have Deming code in your talks, right? So he says, eliminate the need for massive inspection by building quality into the product in the first place. Right. What does that mean for us?

Well, for us, it basically means that we try to build the quality in, meaning before we release it, which we can do every five minutes or every 10 minutes. Whatever we do, we know that it's probably going to work. And then you're like, so, yeah, but what happens to your code reviews? Do you do code reviews, pull requests, stuff like that? A lot of people do you like it?

Say, no, I don't. It slows me down. Right. So we started investigating and investing in full CI. This is part of our pipeline I was actually going to show it live, but it's pretty similar.

This is typically what we do. It's actually running from when I check in my code. I've already run the unit. The unit test on my. Wow, that was close.

On my own laptop. Right. And it goes into the pipeline and we do the format and the linting and prettier goes around. And then we test everything. Sonar does the quality checks.

I'll show you sonar in a sec. We publish it everywhere so it gets containerized. And then we deploy the web test. We deploy to acceptance. There's usually load test here, but in this particular case it's not.

And then we deploy to production. Done. That's it. Right. Pipeline runs for about.

It depends on which of the services it is and how big it is and which of the apps it is. This one run in 30 minutes. So I push it out. In the meantime I can continue. Right.

And then after 30 minutes people can use it in production. For us, that's crucial. Right. So we do this all the time. It does mean you need to do all of this.

Right. You have to have your infrastructure as code. We have our pipelines defined in a piece of script and whenever a pipeline runs, it picks up a definition and then it runs the pipeline. If we change the definition, we add stuff to it. We usually add stuff more than we take out.

We sometimes take out stuff as well. But then all the pipelines after that use that particular new definition. So you need to do this in order to be flexible enough to have your 150 different pipelines working. Right, so next part, testing. As I said, I'm going to do this really quickly.

Right. So I'm going to browse over a lot of stuff. So what about testing? Well, I was on the. Where was I yesterday?

Oh, on the air. Oh, we're on the airport. I wish I had that. There was a guy on the airport. He was on the plane to Oslo.

I think he was waiting for the plane to Oslo and he opened his laptop and he had a spreadsheet open on it. I took a picture of it. The guy doesn't know, of course. And it was a test plan and it contained steps to take in order to test software. Of course, I couldn't put the picture in the deck, but I forgot to as well.

So it has all these steps and then check marks if it was checked. And it takes a lot of time to go such a test script. Right. Too long. Basically, if you want to go to production 50 times per day, you cannot do this.

You cannot have people manually testing stuff. Well, not for everything. That is just for the new features. Look, ah, does the button fit in the right place? Does it work?

Well, if it's responsive and stuff, but that's about it. The rest should be automated rather. Right? So how do you automate stuff? Well, this is the stuff you don't do, right?

This is literally a picture I took from a guy who was on my team for a while and he literally did these tests manually. He filled in, he printed out all the spreadsheets and then he filled them in by clicking on the buttons in the page. You should not do that. It slows you down. It doesn't allow you to move or redeploy stuff in every five minutes.

So take this out. And the thing is, a lot of people still do this. A lot of people have very little unit tests. Well, if you have an 80 million line COBOL code base, you don't have unit tests. There's no unit test framework for cobol, at least not that I know of.

But in all other stacks and platforms and ecosystems, you should have unit test way more than this, right? Because it gives you a sense of security. It gives you the idea, if I deploy this stuff, it would still work because all my 3,000 tests are still green. If you write code in open source and you contribute to open source, this is the only way to go because otherwise you never move forward. We have an open source framework.

If I add stuff to it or I change stuff, I rather don't, but I just like to add stuff, then I need to know that all my tests are green because people depend on it and I don't even know these people, right? So that's the difficult part. So you need to move into. Oh, it goes too fast. You need to move into a direction where you have way more unit tests, automated API tests, integration tests, automated web tests, automated system test, automated load test, et cetera, et cetera.

Very, very little manual test, because this is the only way you can build in the quality in the first place, right? As Deming says. So when it's something, a unit test, I like this. The not definition, the anti definition, basically by Michael Feathers. He said, well, a test is not a unit test.

If it talks to a database, it communicates across the network, or you need some configuration, or you cannot run them in parallel because that also will slow you down, right? So that means every test you write needs to set up its own properties and also needs to break down or tear down its properties after it's run. Because otherwise it cannot run in parallel with other stuff. So you cannot leave stuff in a test database or whatever you do. You need to take it out again too, right?

All those matters come into play when you do this. So he basically saying you need to be able to test stuff in isolation, and that's the easiest way to make this work. It's still hard, by the way. But testing is everywhere, right? If you look at our pipelines, it's here.

It's like we do unit testing a lot. We do syntax validation, we do linting, we do static code analysis, we do API test, security as integration, web test, performer test, Lotus. And to Andy, not a lot, just a few change too often, right? So they change too often. You only want to have a few of them.

So that makes it better to do. So this is what we do. Nothing usual this, but you might start with the unit test. That's the lowest level of having stuff. So, yeah.

So static analysis, this is sonar. So I'm not advertising for sonar, but we use a lot. It's free, actually. And we look at code coverage not just of the existing code base, but also on new code. This is the trick, right?

If you have a very high level of code coverage on your code base, like 95.4%, and you add two lines of code to it, it'll go down to 95.2 or whatever, right? So it's still okay. So you can not have unit tests on your new code for a very long time until you go below your threshold. So that means you not only need to measure it on existing code, but also on your new code. We have 80% code coverage levels on everything.

Not because we, like, annoys us, to be honest, but it's a way to sort of keep the discipline going of adding tests to your code. Not everybody likes doing that. Nobody does, by the way, let's be honest. But it does allow you to code with way more confidence, right? So you don't have to like this poor developer who was on my team.

He was praying for his code to work. That's good, right? Not sure if it helped, but you can see why he's praying, right? He runs his IDE in light mode. Just don't do that.

Who does that? Well, anyway, so we stopped doing pull requests because it slowed us down. That doesn't mean nobody needs to do pull requests. I wrote a post, I think I put it on LinkedIn. It was an article for a magazine about doing pull requests.

It works in some organizations, right? If you are at a large international bank and you have 30 teams working on your mobile app and you can only release it once every three weeks. It kind of makes sense to do this. But for most, well, let's say smaller teams or smaller configurations. I'm not sure you should actually do this because what happens with a pull request, right?

You write some code to solve a particular issue, you push the code and then you require a pull request, or you do a pull request and somebody else needs to do the code review, that somebody else might not be there, right? They might have a day off. By the way, if I would check in my code today, this is a national holiday in the Netherlands today and I'm not sure why you guys are all here, but we have a day off and that means nobody will do it. So because it's Thursday and everybody has a day off, they'll probably take the day off tomorrow as well. So that means if I push my code now or yesterday, late in the afternoon, there is nobody going to look at it until Monday, probably in the afternoon.

That's almost a whole week, right? That means if they give feedback, usually on the linting or the formatting or whatever, those which you can automate, usually it's not about your code or the architecture because nobody has the time to go through all the code you're writing. Especially if you do larger pull requests, which you shouldn't do, by the way. And then it becomes confusing because you get it back a week after and you're like, what? What was I doing the week before?

Hey, I'm doing something really different now. So you need to start context switching. Context switching, as we all know, is very expensive. So that's why pull requests usually slow you down. So in a very regulated, compliant organization.

Yes. Not in an E commerce company, we don't do it. So we don't do code reviews. Right. We just do trunk based development.

What? Trunk based development, can you do that? Yes, and you probably should. So the thing about trunk based development is if I pull my code from the repository, I make a change and I want to push it back because we all push the trunk. That means I want to push it back as soon as possible, as soon as I can.

Because otherwise I get merge conflicts and I don't like merge conflicts. I never understood how these merging tools work. It's a mess. I'm too old for this shit, right? So I'm used to not having version control systems.

I started writing code when the only version control system was I locked the file. So you cannot save it. Literally, right? So that's how I started. But here's the thing.

So if I do trunk based development, I want my code to be back into the repository as fast as I can. So you start going through smaller and smaller, smaller changes because you want to be the first to check in the code again, right? And then you pull again and you, etc. Etc. So it makes you.

Or it pushes you towards the direction of making stuff really, really small. And really, really small again is easier. That's the way it is in software development. So yeah, data, I have two more things and I have four more minutes or something like that. I have no idea.

When we start, we start a little bit late, right? Oh, goody, goody. So I can talk for another half hour. No, I'm just, I'm just kidding. So.

Oh yeah, yeah, no, I'm going to try to do this in like eight minutes. So it's an estimate, right? We're software developers. I don't know how to estimate. So it could be 20 as well.

So who wants the data? That's the next part, right? Two more things. It's data and a way of working. So if you have a monolithical application, it's usually like this.

I have this application, it might contain a domain, it might not, but it usually has a database that sort of like. Or basically the application mimics the database. This is what a lot of people write their code to write. And it works in a monolithical way, but it gets complex as soon the model gets more complex. So then you think, oh, let's go to microservices.

So you know what you do? The first step is we break out all these services. And that's fine, but because we don't want to make it too hard, what we'll do is we'll still talk to the same database. This is pretty hard, especially if you're running SQL databases or relational databases, because everything's connected to each other and you still have the dependencies here, right? So here's my dependency chart again.

Dependencies again will kill you even if they're in a database. Not anymore in UI because we split it up. Not anymore in the domain because we made all these services. But the database is still going to do this. So you might want to go in a direction where services own their own data.

And you're like, ah, that's cool. It's also complicated, by the way, because it gives you a lot of interesting issues, like what if I want to get an order out? So I talk to the order service, but in the Order, there's products. So what do you do now? The interesting thing is you don't talk from one database to the other.

Right. It's the first rule of microservices Club. You don't talk to others. No, that's bad joke. Anyway, so you have to talk to the product service.

Or alternatively you might have a snapshot version of your order in the order database. That is with an order, usually the case. That's what we do. So we make. When somebody places an order, we make a snapshot of everything related to that particular order because that is the state that we want to know it in.

We don't want to say, hey, the price of the product changes. So my order is changing as well. No, it doesn't. So every situation that you encounter in your domain needs a decision about how to deal with the data and where to keep it. And redundant data, duplicated data is very much a thing.

If you do microservices because you want to save it for different reasons other than having a fully normalized data model. We actually don't even use a relational database in our. The good thing is if you put stuff in different, the services own their own data. That also means that you can store it in different ways. Not everything needs a relational database.

Lots of stuff can do without it perfectly. You can get your data from another system from whatever authentication provider you might have. We use Firebase or some stuff is very hierarchical. So you put it in a relational database. We use authentication document databases for most of our services because document databases have the nice characteristic that the whole aggregate that you have inside of service, you can store it in a single statement.

I just stored adjacent into my Mongo database. Done. If I get it out, I get it out with one call. It's the same call I used to do a request to my service. So it's pretty similar.

I'm going to skip all the blue boxes here just, just for a bit. So yeah, go into the services own their own data thing. It's the last thing you need to do to break down all the dependencies in your domain. One thing left, ways of working. So I've been doing microservices since like 2014, 2015 ish, something like that.

And we realized that the ways we work became very different to how we used to work before. And it. It's under the influence of the fact that you want to go fast and that you can go fast and that you can release the production 50 times per day. That makes you rethink how you work. Did you see Ken Beck's keynote this morning?

Yeah. He talks about forest and we live in a forest. The rest of the company is still a desert, but we are sort of like, I don't know what, the oasis in the desert, I suppose. So we very much aware of the fact that there is no individual code ownership. You cannot do that if you have 13 people and 160 repos.

Right. It doesn't really work because you need to be able to get to any part of the code and change it if you need to change it. So that's what we do. Everybody can touch every piece of code. We usually work together quite well.

Right. It's not that we all individual have their own set of repos or their own services. No, we don't. We just talk to each other the whole day. That's nice.

Right. So that means that if we want to do this and if you want to go to a way that you can continue to deliver value to the business 50 times per day or even 10 times per day or once a week, doesn't matter. Right. You need to have close collaboration. Here's my team, all working individually.

We rarely do that, by the way. So we do a different part and we have to look at autonomy. The thing is that I think that. Well, I'll show the next slide, which is the interesting slide for this, actually. So this is De Hock.

He was just the CEO for Visa, and he said this. And this is the second thing to remember from this talk. It's this quote. He says, simple, clear purpose. Right.

Ken talked about purpose as well. And principles give rise to complex and intelligent behavior. If you want people to behave intelligently, you give them the space to do that. You give them the space to experiment, to make mistakes, or to have an error and then come back from it. And if the next bus comes along in five minutes, it's not that bad if somebody makes an error.

On the other hand, complex rules and regulations give rise to simple and stupid behavior. People started following processes. They behave the way the process describes. Right. In a microservices world, that is not, to my knowledge, the best way to do this.

So here's the trick. The trick is to simplify, to amplify. I love this quote. This is great. Right?

Simplify everything. We pushed everything out of the way that we didn't like or found necessary. And that is way more than you might think. This is a crossroads near where I live in Amsterdam. At some point, the city council decided to remove most of the traffic lights.

There's A few left for the trams. But trams are too big to take into account, right? So they removed all the traffic lights and everyone's like, what? How do we deal with this? So the Netherlands is a very much over regulated country, right?

We have traffic signs, like every street has to have at least 50 traffic signs. There's no other way to do this. We have. It's all very much regulated. That means it's the simple and stupid behavior that you're going to create.

But if you want people to think, you take away the rules, so they remove the traffic lights on this crossroads. I usually come from here, from here, and I go straight here, diagonally. Nobody likes me doing that, but usually. So what happens if you do this? If you take away the rules, what happens?

People start to communicate. You actually start to look the other people on the road into the eyes, see what their intentions are, what they want to do, and then you decide to go first Anyway, I live in Amsterdam. We do that, right? We have no rules. But that's the thing.

You start communicating with the other people who are in the same context and that makes you aware of their intentions, right? So you look into the pedestrian's eyes and you see, oh, I'm gonna let them cross because, well, they're slower, but I'm not going too fast. So go first. And you just nod and they know and they go first, right? And it works even with cars, except for BMWs, of course, but they're sort of like the exception.

So you take away the rules and people started to communicate. And if they communicate, you can do better work. So this is my team again, right? So this is what we don't do. Well, wait, basically I'm going to skip this one.

So we don't have scrum. That's been, I wish for me for years. I stopped using scrum 15 years ago. I don't like it very much. But that doesn't mean we do Kanban, right?

It's not that if you don't do scrum, you do kanban. There's like 50 shades of ways of working between and maybe even 500 ways of working in between. We don't have sprints. Sprints don't make a lot of sense. If you push the production 50 times per day, right?

There's no reason to do this. So we also don't have reason retrospectives. We don't have a Scrum master. We also have no product owners. Well, we do have product owners, but everybody on my team is their own product owner.

They do that. I'm going to talk tomorrow a bit more about my team, but that's the soft side of it. And we have no users. We don't estimate except for ballpark figures at a very high level. That's about it.

We're experimenting with fewer standups. That means we don't have a stand up on Thursday when we're all in the office. That's our office day. It's nice. So we don't have pull requests, no code reviews.

We have very simple architecture and that's basically how we roll. So the stuff we do do is we do event storming when we need to figure out how processes work and how the domain is set up. Works quite nicely. Fits really well into DDD and into microservices as well. We do a lot of.

Well now my clicker gave up. We do a lot of pair programming. They look a bit depressed, but this is Eugene. And Eugene always looks depressed. So he's always worried, right?

It's in his. In his blood. Except for all the beer, but. And so we do a lot of mob or ensemble programming. It's.

It's stuff that goes. It's dynamically right. We don't say, oh, we're going to set a mob up for. For now. We had a serious breakdown two days ago and we, well, we were all there basically.

Nobody had to say please come into the call. We do them. We do the huddles also remotely. So it works. And we do of course vibe coding.

You need to do vibe coding, right? Otherwise you're not hip these days. So we do that too. If you don't know what it is, look it up. Or there's probably a bunch of talks about it anyway.

So yeah, now my clicker is like empty, I think there. So this is the thing. We try to solve a single problem every day using a very simple way of working. And we realized that no two items that come off a board, whatever board it is, actually require the same set of people. So we started saying, you know what, we'll just throw stuff from the board and figure out what you want to do.

So this is the recipe that we sort of use. Again, this is not a rule. It changes all the time how people use this, but it's like somebody's done with what they're doing and they'll pick for many of the boards that we have, we have three actually. They'll pick up something that they like to do or they're good at or they want to learn about or that they have no idea how to do. But they want to figure it out because they want to do something complicated.

It's all fine. It's up to them to do that. Up to us, basically, because I do the same. Right. And then usually people come along and they sit along and they work together for a bit and then they push it to production done and they go to the next item.

Some of these items are really small, like a half an hour, sometimes two hours, sometimes a day or a day or two, usually much bigger. We have a few that are really bigger, but I really hate them actually. So we work in what we call micro teams. Well, everything was micro. Right.

So that was the point. So. So this is a developer and an ops guy. I don't remember who's which, but they both could be Java developers because they have beards. This is my team, actually, currently.

The other ones were also from my team previously. Francisco using this front end. So this is probably some backend stuff because he looks a bit weary. And this is two Java developers and a tester. You can guess who the tester is, of course.

Yeah. This is a Java developer without a beard, by the way. And they exist in real life. And this is an architect and they develop. Right.

The developer already knows it's not going to work. The architect's still praying. Yeah, that's their role basically. Right. So it's a very organic way of working.

Nobody sets any rules for anybody else outside of the architecture. And the way we do CI CD and the way we write our tests, that's basically the boundaries we have all other things, people on my team figure it out themselves and that really works well in our case. Right. So in retail, I'm going to do this really quickly. So yes, microservices really helps you solve problems, but you need to start doing it for the right reasons.

Right. Don't go for the scalability if you don't have scalability issues. As an example, there's lots of wrong reasons to do microservices. Usually when you do it for the wrong services, it's not going to work out. It's probably also the other way around.

If it doesn't work out, you're probably doing it for the wrong reasons. I don't know, I need to figure that out one and then you need to take into account lots and lots of stuff. It's not an easy road to travel. Right. So you need to really have a problem that it solves for you and then solve a single problem every day.

Right. And basically simplify to amplify. Right. That's the whole key is you take away everything that you don't need. Experiment with it, right?

Take away parts of your ways of working. Just figure out if that works for you. If it doesn't, put it back in. That's fine, right? But allow yourself to do those experiments, because that's the only way to move forward, right?

Use the experiments. And you can never stop learning at Sauer Industry. So it's really good that you're all here and of course I hope you enjoy the rest of the conference. But we're also in the best industry in the world. There isn't a job I would rather do.

My youngest son once asked me, said, dad, if you were not a programmer, what would you do? And I was like, I would learn to write code as fast as I could. And he said, ah, makes sense. So that's the thing, by the way, if everything else fails, you've tried everything. There's only one thing left to do, and it is you remove your node modules and you do an NPM install.

Thank you for being here. I'm sorry I'm running late. Not sure how much late, but. Sander, thank you so much.
