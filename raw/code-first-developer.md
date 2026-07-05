---
url: https://khalilstemmler.com/articles/the-phases-of-craftship/code-first/
date_fetched: 2026-07-05
backfilled: true
---

# The Code-First Developer

"Wait, is that Sean? Hold on a sec," I exclaimed, interrupting a friend mid-conversation.

I had just seen him speeding past the famously cursed McDonald's on the corner of Queen and Spadina in downtown Toronto: Sean, the programmer who had opened my eyes to what great code could look like.

Walking alongside a fellow Software Essentialist Course member, we strolled past the McDonald's when I spotted him — this key developer who had played a pivotal role in shaping my career.

"Whoa, no way... Sean!" I called out, quickening my pace to catch up with him.

Sure enough, it was him.

"Oh, hey Khalil, funny running into you down here. How's it going?" Sean greeted me warmly.

In that moment, memories flooded back, reminding me of the time we crossed paths in 2017.

I had been working at a small frontend consultancy, seeking a job that would allow me to focus on my startup, Univjobs, in my spare time without constantly lingering thoughts of work in the evenings.

I was assigned to collaborate with Sean on a magnetometer software project, exploring the depths of ocean research and exploration.

Until then, my experience had been marred by messy codebases no one wanted to work with. My first job involved:

- Injecting SQL code directly into a massive Java file.
- Totaling over 2000 lines.
- Determining its correctness based on whether it returned any data.

Welcome to the industry.

Fast forward to that consultancy where Sean and I crossed paths. During a lunch and learn session, Sean prepared a talk about Domain-Driven Design. I could feel the cogs in my mind change irreversibly.

Assigned to work with him on the UI portion of an application, I was astonished by the elegance of his C# code. Despite the complexity of the problem we were tackling, I found myself grasping the essence of the domain simply by examining his code.

But something didn't quite sit right with me while at that consultancy. The conversations between other consultants seemed… way too surface-level.

My colleagues argued about things like what Dan Abramov said about single-file React components and why Vue is better than React.

It felt like we were stuck in a repetitive cycle, building the same app for the same 2 clients that funded 80% of the business year after year, just continually refining the implementation with the latest libraries and frameworks.

It felt like the kind of cautionary situation Robert Greene or Patrick Bet-David would write entire chapters about.

I couldn't help but question the value we were truly providing. Many of the other frontend developers were deeply curious and wanted to learn backend, but instead of taking that into their own hands, I noticed most were silently waiting for our employer to fund their learning efforts.

I never expected that to happen, and guess what? It never did.

Instead, for the next 8 months, I spent my mornings and evenings writing buggy backend code, trying to learn how to make it better.

To some, that might seem extreme, but I really wanted to figure it out.

And after everything I'd experienced thus far in my college, university, and early professional experience, I had very little trust that things would just get sorted out by themselves.

And during that time of work & study, the industry certainly did shift.

In a few years, the advent of AI was upon us. With GitHub Copilot and ChatGPT, gone was the inherent value of being a mere code-first frontend developer.

Laws from timeless texts like The Mythical Man-Month (1995, Fred Brooks) such as the law of least developers, whereby "adding more developers to a project typically makes it take longer" - well, these laws locked into place.

Suddenly, the value of a single full-stack Pattern-First Developer with access to ChatGPT became greater than 5+ frontend developers all combined.

From 2021 to present day, I've seen a ton of low-skilled frontend and hyper-specialized developers lose their jobs due to this phenomenon I've been calling the Value Differential - the teeter-totter of what's valuable and what isn't (more on this later).

Back to Sean.

With him in front of me, I asked him what he was up to these days.

When he told me the consultancy we used to work for died and that he was being made CTO at the client's company, I was not surprised at all.

Excellent work always prevails. Mastery is truly recession-proof.

At some point in our conversation, we spoke of Bruce Lee.

"You must be shapeless, formless, like water. When you pour water in a cup, it becomes the cup. When you pour water in a bottle, it becomes the bottle. When you pour water in a teapot, it becomes the teapot. Water can drip and it can crash. Become like water my friend." — Bruce Lee


That encounter with Sean on the street brought back a flood of memories, reminding me of the path I had taken and the lessons I had learned. It solidified my conviction that mastering the first principles and possessing key mental models in this evolving landscape is critically important because it simply is not enough to be a mere Code-First Developer anymore.

--

In The Software Essentailist, we walk through what I call the 5 Phases of Craftship:

- Code-First
- Best Practice-First
- Pattern-First
- Responsibility-First
- and Value-First

Each of these phases marks a distinct shift in your value system as a developer, how you think about code, and how you decompose and solve problems.

Because definition creates your *perceptual* reality, the more you refine your thinking (ie: your definitions, beliefs & knowledge about what code is and how you decompose and solve problems with it), the more complex the problems you can confidently decompose and solve.

Let's discuss the first Phase: the Code-First Phase.

*(I sometimes call this "The Pain Phase", the "Imposter Syndrome Phase", the "First Phase", or the "Necessary Phase" of craftship).*

## What we'll discuss

In this short article, here's what we're going to do:

- We'll do a quick recap of who the Code-First Developer is
- We'll review the traps that prevent you from progressing
- Learn the next steps to transition from Code-First to Best Practice-First

## What is the Code-First Developer?

Alright, so the Code-First Developer.

Who is this again?

Ultimately, this is the type of developer that has learned to code but has yet to learn to craft.

As we discussed in a previous module, the 4 pillars that every developer needs to focus on design, testing, architecture, and strategy. We call these **The 4 Pillars of Software Development**.

And these 4 pillars have a chief purpose.

And that is to act as assistive tools for us to help the 3 roles (customers, developers, users) achieve their goals.

What are all those goals you might ask?

In essence, you need to learn how to work in ways that will prove to be valuable. If you do it right, when you work, you'll keep the 3 roles in benefit by either creating:

- TimeGivenBack (to pursue other human desires)
- or TimeEnriched (to enjoy the present moment)

That may be too abstract, so let's look at an example that should make sense to you.

We'll use the developer - actually a frontend developer since when we're at the Code-First phase, most developers tend to be frontend developers.

So, what are some common problems for frontend developers at the code-first phase?

What might you be asking yourself?

- how do I write E2E tests that aren't flaky? → because it takes time to write these, watch them run, and I hate the feeling of having to go back and re-do them every time the smallest thing changes
- how should I be writing unit tests? → because I've tried to figure it out, but it was confusing, and taking too much time to know what to do
- what exactly should I test in the first place? → because I literally have no idea, and I know tests are important but it doesn't seem very useful to test something like Redux
- how should I organize my code so that it's scalable? → because I notice that I spend a lot of time flipping between abstractions and others seem to not really know what to do
- how do I do things the Angular / React / Vue way? → and so on...
- how can I get better at architecture? I don't know how to use in a testable way? → and so on... 

Key questions.

Each hints at a tiny slice of pain (wasting time and feeling frustrated).

And each with very little good answers at this level.

That's because when you're at the code first phase, you don't know much of what matters.

Yes, you can use the basics of your tools to solve problems but… testing, design, architecture, strategy? The stuff which actually allows you to keep those 3 roles in benefit? Not likely at this phase.

We're talking about understanding the skills that allow you to:

- Make testing fun and enjoyable?
- Approach your work consistent?
- Adapt to the changing requirements flexibly?
- Select a pragmatic testing architecture that will allow you to feel confident in your code given the time constraints?

These thing are really important. But they don't come until later phases.

So we better move through 'em.

## The Value Differential has changed

I won't spend too much time on this here, and you already know, but being Code-First used to be fine.

Thinking back to the early days of React, when things were kind of a mess, the API was changing every few weeks, and Webpack was the worst thing on the planet… this was when the most valuable thing you could be WAS a React Specialist.

Back then, in 4 to 6 months, with access to FreeCodeCamp and some 12-hour YouTube video compilations, you could possess an incredibly valuable, in-demand skill nearly everyone needed.

But now?

What do we need now to get the same result?

Internet access and a ChatGPT account.

Yep.

"Hey, ChatGPT, can you generate a React Hook that helps me synchronize XYZ?"

Sure, here you go.

Thank you, artificial intelligence.

So what does that mean for all the freshly graduated frontend bootcamp devs looking for jobs?

It means we need a different strategy, of course.

Your biggest problem is the sheer amount of unknown unknowns.

When I backtracked the path I was walking in solidbook, I realized that the biggest problem all Code-First developers face is that you don't even know what you need to know, and you don't know how to BEGIN learning what you need to know.

It's the wild west.

If you listen to what everyone else tells you to do, you'll learn new libraries and frameworks every few weeks. NOT leading yourself towards their depths and getting to the next Phases. New tools.

## The Expert Junior Developer

This trap of taking a breadth-first approach keeps you stuck at the surface and can potentially turn you into the Expert Junior Developer.

You may have been around for quite some time, but like many of the consultants I worked with in the past at that now dead frontend consultancy, when you get back into the industry, you're still asking yourself:

- how do I write E2E tests that aren't flaky?
- how should I be writing unit tests?
- what exactly should I test in the first place?
- how should I organize my code so that it's scalable?
- how do I do things that Angular / React / Vue way?
- how can I get better at architecture? I don't know how to use in a testable way? 

These are the same questions.

And we all know what comes next: burnout.

## Breadth > depth leading to burnout

I interpret burnout as when the level of effort and energy you spend doesn't match the level of fulfillment you should experience.

I believe the key to lifelong fulfillment is Mastery.

To be able to set a goal, chart the path, and consistently walk that path confidently, regardless of what it pertains to, is one of the most fulfilling experiences, in my opinion.

But the key, as I've probably said twelve times in different ways, is depth, not breadth.

Do less, more.

🌀 Focus on the main idea: Responsibilities. The metaphysical, atomic unit of matter that describes what software is comprised of is called a Responsibility (for doing and knowing). Let's continually refine our understanding of this. (You can take a look at Responsibility-Driven Design if you'd like to jump ahead a bit).


## How to progress to Best Practice-First (what to do instead)

So let's discuss what we gotta do to move toward the next phase.

### Step 1: Shift your focus to Responsibilities instead of Code

We want to get to Value-First so that you can do life-changing stuff.

But doing so involves developing a deeper, more mature perception of code and how to code properly.

First step: start to pay attention to the various responsibilities your tools handle.

Start to notice the things that your tools do and know.

What are the things that your web server does and knows?

What about React? What does it do and what does it know?

It's funny because you can use React every day for five years and become an expert, but when you expand your perception of what it is you're dealing with...

When you no longer see React as the thing you use to "build the frontend"...

But instead, you see it as a tool which allows you to "build the UI while coding outside-in" @ the Best Practice-First Phase.

Or you'll see React as the tool you use to "build your View Components and Page Components" separately from how you organize other concerns using more robust frontend patterns like View Models, Domain Objects, Gateways and so on.

### Step 2: Commit to becoming a Value-First developer (which means you have to become a fullstack developer)

You want to be able to produce as much value as possible.

That means you need the ability to create information systems from end to end.

But not just any information system. I'm not just talking about using CRUD and REST.

These need to be information systems that are Software Artifacts of Value.

This means they help users, help customers achieve their goals, and developers can carry out their common development use cases.

What you place your awareness on, you manifest.

Let's focus on that.

It's fun!. The world rewards you for that the value you put out.

### Step 3: Build an information system from end to end, making all the mistakes

In the first Phase of Craftship, you want to focus on building information systems from end to end.

That means frontend and backend.

Build a React app that talks to a backend and a database.

Build a Gatsby or Next.js app that uses a CMS.

Doesn't matter. Just build something that a real human being can log onto an accomplish a task with.

While you're building it, don't worry about best practices or if you're doing things right.

Just make it work.

Make it work and survey the landscape of tools you needed to use along the way.

How do they help you cross the gap?

### Step 4: Validate your understanding

As I said, the main challenge at the Code-First level is unknown unknowns.

Focus on building something - anything - along the lines that it could serve others and a customer could possibly benefit from it.

And while you do this, zoom all the way out, and then focus on what your tools are really for, why you use them, and what they do well.

I will give later give you some strategies for ensuring you understand the role your tools play, the responsibilities, and how they either help or hurt the three roles in achieving their goals.

## Summary

- The Code-First developer might be technically skilled at the implementation side of things, but the Value Differential says that code is not enough.
- The biggest challenge Code-First developers face is the sheer amount of unknown unknowns — knowing what to learn, what to prioritize, and how to learn it effectively without suffering from overwhelm or burnout.
- The goal is to be able to think and work responsibility-first because that's what will allow you to focus on what matters (creating value for the three roles) later on.
- So, let's take it one step at a time. To get to the next Phase of Craftship, you must commit to becoming full stack, build a real, basic information system that does something valuable, and ensure that you understand the purpose of the Code-First tools and topics used to do so.

Not sure where you are on the path? Take the Phases of Craftship quiz — 7 questions, personalized results.

### Stay in touch!

Join 20000+ value-creating Software Essentialists getting actionable advice on how to master what matters each week. 🖖

View more in The Phases of Craftship
