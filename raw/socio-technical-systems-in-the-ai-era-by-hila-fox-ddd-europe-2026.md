---
url: https://www.youtube.com/watch?v=5BFTtYKaRJs
date_fetched: 2026-10-02
---

# Socio-technical Systems in the AI Era by Hila Fox - DDD Europe 2026

- **Channel:** Domain-Driven Design Europe
- **URL:** https://www.youtube.com/watch?v=5BFTtYKaRJs
- **Duration:** 49m 27s
- **Transcribed:** 2026-10-02

---

Hi everyone. So first of all, I'm very, very happy to be here. A lot of lights. Okay, so we'll get started. We're going to talk about socio technical systems in the AI era.

A little expectations, warning. So I can't predict the future and basically this talk was built to be like a thought provoker. Okay. It's not like I have all the answers. I can't predict the future as I said.

So I'm going to carry on and maybe afterwards we can, I don't know, go out and fight and talk about things that you disagree with a little bit about the structure. So there's going to be a short intro. I'm going to talk about how we mirror agents, how we build agents today and we mirror them in our shadows. Afterwards, going to talk about the transformation that the companies that were working at, the transformation that is happening. Can't leave out the humans.

We're still here. They didn't manage to replace us, not just yet. And how we put all of this together. Okay, so let's start with the intro. So first of all, definitions of technical systems, the recognition that technology and human organizations aren't separate things, they're one interconnected system where the way people work shapes the technology and the technology reshapes how people work.

Cloudsonnet 4.6 very, very smart individual that gave me this definition. Awesome. So in socio technical systems we have actors, right? So we have human actors and we have non human actors and we have boundary objects. So I don't know, we can say boundary object is like the object that connects the different domains in our systems.

Right. If we could imagine, what do I call it? Like a patient entity in a hospital. Right. It has a lot of different meanings and the human and the non human is pretty obvious.

But claim number one is that AR agents are not just tools, they're not the technology part. There are actually new socio technical actors because obviously they're an artifact, an artifact like software, but they're also reasoning. So if they are able to make decisions instead, instead of humans, where do they fall in this diagram? And now this thing is here when we need to know what to do about it. A little bit about me.

I'm a product manager, but I mumble every time I say this because I was a principal engineer until two months ago and I turn to the dark side and now I need to apologize every time. No, but I'm still a technical person. I still know what I'm talking about, believe me. So this is what I do. I come from a Background of technical leadership, product leadership.

I do public speaking, hence I'm here. By the way, just so you all know, DDD Europe was my first international conference a few years back. So for me it's very closing to be here as well. DDD enthusiast. Even before AI I was talking about DDD and now I'm talking about how things connect together and, and I'm into board games.

The right amount. My partner for example, she's not the right amount. If you've seen like these houses with the big closets of board games. We can talk about this later as well. And I think my perspective is unique when coming to talk about SOCI technical systems, DDD and AI because I work at a company called kodo.

We do AI code review. And it's not a marketing pitch but I'm going to use the company for examples. And also it's sort of I build AI using AI for AI for companies looking for AI. So it's sort of a lot of different perspectives into the same topic. So it becomes pretty interesting.

Does it say there I'm gonna move this? Okay, so a little bit about kodo because I'm going to use it for examples. It's a multi agent code review plat. This is the architecture in general, right? There's like shift left capabilities like Skills and MCP and ide.

We also have a code review that is installed on the git providers and underneath this there's like a large context engine and a lot of different enterprise features. That's about it. That's about very roughly the architecture. Right. We have several central services.

The biggest one is the code review capability and, and a lot of different interfaces going around it. So this is us just for context. Awesome. So let's kick it off the org chart. Right?

Such a pretty diagram, isn't it? So neatly drawn here. But we all know that org charts are a lie. It's not really how things happen in reality. It's in reality, let's say Jane, to be able to perform a report, she needs to talk to Emma.

She doesn't have to go through these lines. Right. Or for example Michelle and Alan. If they want to complete a project, they have to collaborate even though they're in different departments. And sometimes you need to have a cross organization project just to give a deliverable.

This is the reality of our communication patterns. And we have Conway's law. Right. So Conway's law, any organization that designs a system will produce a design whose structure is a copy of the organization's communication structure. So if this is what's happening in reality.

And this is the sentence let's say we believe to be true. So if this is how it looks like, right, And I want to come and say, okay, I'm just going to drop an agent here, here, here and here, here. This is where I'm going to drop an agent. And I'm expecting everything just to work, even though I'm trying to replace specific squares in my diagram. Well, every once in a while I'm going to have a very, I don't know, stupid picture and I'm going to give you the prompt that I did with GPT to create it.

Because this is really ridiculous. You can just write very silly things. Very funny things come out. So it doesn't work out of the box. Okay, so let's talk about mirroring.

Because in reality, this is what we're trying to do. We're trying to just build agents that mirror the actions that we do as humans. Right. So what do we do as humans? As humans, we write, we create, we organize, we analyze.

And can anyone think of any one of these actions or things that you do in the day to day that you like complete a task independently without talking to other people? No. Yes. No. Yes.

You're not very participating. I mean you can like more energy, you know, like, it's fun. We're having fun here. Yeah. No, I never complete a task of my own.

Yeah. Okay, great. Awesome. So this is that. And the thing is that to complete our tasks we know in our brains, right?

Because Jane knows in her brain that to complete the report she needs Emma. And Brian knows that to complete his deep work task, he needs a lot of different things. And Alan knows that he needs to talk to Michelle to complete his cross team work. Right. So we know all of this, but as I said, we're trying to just like throw agents at it.

So what's the gap? The gap is the context, right. We need to actually have all this data somewhere for the agents to be able to work with it. So we're building agents to automate the actions we already perform. Right?

Everybody are talking about automation, utilization, all of that. Very nice. So from what I see, a lot of the conversations are around giving agents enough power. Everyone are worried it's not going to be high quality enough, good enough, going to do this. So we're talking about this, but we're not talking about the problem of the hidden human element, the human sensitivities.

So for example, Karen and David are, you know, they're going to block the work that you're doing right if you're not going to come to them, because that's how Karen and David are. These are how humans behave, right? And you can't handle this. They're in the company for 15 years. Either do it or don't get it done.

Or, for example, Tom was a domain lead for three years and he moved to a different team. But still Tom insists on being part of all the decisions. And this is the reality of humans and how companies really work, right? You know, so you send him a message when you have something to do and then he responds. And, you know, we give a good pat to his ego.

He says, yes, we carry on and work is being done. But claim number two, we are automating our organizational illnesses into our agents. How are we going to automate this type of process that I know, what will it say? I mean, so imagine the point in time that you're building the agent. What will it say?

It will be something like, you have to write a slack message to Tom because Tom gets super defensive if anybody makes decision without him in the loop. Well, you can't write that, you know. And Kat Wu, she's the head of product at Cloud Code in Entropic, and that's a quote from her from a podcast that I've heard. 95 automation isn't good enough. Because if it's 95% automation, it's not an automation, right?

And we can't keep in people, especially people that strive to keep some sort of power in the loop if they're not really meant to be there. So what happens is that all these hidden communications become your AI DNA, right? Because the path of least resistance is to build automation for the processes you already solved today in your company. It's a lot harder to say, okay, let's take a step back. Let's look at this process.

Is it behaving as it should? Is the steps as they should be? The answer could be no. But if you're not asking this question, what ends up is like, maybe the C suite are coming and they're like, use AI. And you're like, fine, let's run, let's build an agent.

But suddenly I need to tell the agent, I don't know that Tom is in the loop, right? So what ends up. What ends up is that we have system fragmentation, is that organizational silos become agent silos, right? And organizational restrictions become decision limitations. And lack of ubiquitous language is AI miscommunication because the AI doesn't know that you should be better and avoid the silos or you want to use the AI to be able to give it autonomy and automate something.

So all of these are blockers to reach the dream that everyone are talking about, right? So we have the system fragmentation. There's also cultural patterns, right? Sometimes these things are inherent from a company and it's either good or bad or it should be or should not. So for example, if a company is high or low trust, it will have high transparency in decision making.

And this can be context, right? And high risk tolerance can be how much can you move, let's say, of a decision from a human to the agent? And you know, it doesn't mean necessarily it can be either. Maybe it's a company in finance or health, and then they have to be to have low risk tolerance. Maybe the CTO is a little bit cuckoo, you know, and then you have to be low risk tolerance as well.

So. And lastly, also informal networks, companies that have a lot of informal work, things that are happening under the radar just happen to also be very heavy with shadow AI. So all of those things are being baked in. And in the end, these hidden patterns, this, our culture becomes something permanent, right? Imagine I'll give this as an analogy.

Let's say your VP Marketing and your VP Sales are not friends, to say the least. Both of them are like, we're going deep into AI. Both of them are building agents that give prices to customers. The answers are going to be different. They because it's going to be different infrastructure, maybe even worse.

It's going to be different context if they're really not collaborative. So we're going to have competing agents. And maybe something I didn't touch on is also the hierarchical agents. So we have power dynamics between people. Well, I can say that my agent is better than yours and there's nothing you can do about it because I'm more powerful in the company.

These are things that can happen, right? So in the end, all the organizational illnesses come into this. Okay, so claim 2.5. We are automating our organizational illnesses and they are making bad decisions. And it's not the agent's fault in the end, it's our fault because we told them to go to Tom and ask Tom to give an approval before they can move on.

So what do we do about it? I'm not going to go deep into things. I'm going to give two small examples from where I work on those things and some maybe leading questions because this topic is too wide and it's too unique per company. So how do we stop the mirroring. So for example, inside of Kado, back in the old days, you know, like in the AI timespan, like three months is two years and one year is a million.

So when the company started, we were focusing on code generation, not code review. So this is a bit of legacy, but basically what we had, we had an IDE agent. This was the first interface that we have invested at and built co generation agent in there. But tail is all this time we had a different team. They decided to build a cli.

The CLI performs code generation as well. They built a different agent. Now we have two. Two of them are competing. They're doing the same, the same task.

It was also a known decision because the IDE team was our primary interface and we wanted to experiment fast. So we did something on the side with the cli. In the end, you need to consolidate things. Also inside of Kodo, we have, we can say a lot of shadow AI. We do have a lot of duplications.

We use our own tools, we use other companies tools. It's sort of like weird to be in this cross section. We use also a lot of other different AI tools. So how to find the smells, A small cheat sheet. There might be more questions for me.

I find these good. How frequently do agents contradict each other? If you have a situation like that, something that you have inside, I mean is a tool that you're using in your company. Let's say say someone built an automation either for support or for marketing, or for co generation or code review as well. A lot of companies are experimenting with this internally.

Who owns an agent? Can you answer this question? This is sometimes very hard because an agent is a workflow and it answers the process and sometimes it's very wide. So I'm not going one by one. It's for you.

If you want it, you can take a picture. Moving on. Awesome. So I talked about the mirroring, which is I think the thing that we don't talk enough about, right? How we take a step back and how we identify that we're really putting agents into our systems the right way and not just in the path of least resistance.

Okay, so let's talk about transformation. Eric touched about it as well. You know, things are changing, so let's dive into this. So once again, let's think of things we do as humans, right? What is transformation for us?

Well, at least it work, right? So we restructure, reorg, reorganize, we hire new people, right? If we bring new people with new skills to the company, that's transformation. We learn new things, we optimize for costs, we can optimize for other processes. So we do a lot of transformation all the time.

We're always in the move. So I think the most, let's say the thing that is most talked about is the layoffs, right? And the restructuring. And there is a lot of headlines. And actually I built these slides.

The original talk, I think six months ago, I had different headlines, same thing, right? Blah, blah, company fires, blah blah, people for AI. Okay, this is happening. And there's a lot of very scary titles. But if you'll go and read under the hood, you'll see that a lot of the layoffs are actually companies optimizing just to put their powers somewhere else, right?

They want to invest more in AI infrastructure. It's not that they're necessarily downsizing. If they're downsizing, something else is happening with a company. Because if you look at the, let's say The Technology Edge, OpenAI Entropic, all of these companies, they are hiring. They are not reducing their size.

So someone is reducing their size because of AI. Okay, let's just wait and see. Okay, So I wanted to start by touching on the hard part of transformation. Let's see how it goes. And I call this Conway's law feedback loop.

So it looks something like that. And let's say LLMs came into our lives. 3, 4 years GPT 3.5. Surprise, here we come. And suddenly a lot of friction and everyone are trying to understand what to do with it.

And we are sort of here. The transformation is already happening. When I build those slides, this thing was just a dip because I don't believe this is over. So we are here, so we are experiencing this. And the transformation is happening.

And it's painful to be part of it. At some point in time, we will reach a new world order. Okay, either the technology will plateau, maybe the costs will make things plateau, I don't know. But things will happen. And it's also very related to what Eric said.

You can't predict the future. You can only handle what is going on right now. And then, okay, new tech will emerge, new things will happen. Maybe it's AGI, maybe it's small models. I don't know what will disrupt us next.

And then we will go and do it all again. So what about this? So all of this change, we need an anchor, right? And if I would say in ddd, our anchor can be the bounded context, right? So also here in the world of AI, everyone say context is king.

But I'M here, so I'm going to say context is queen and this is my queen of the context. So basically claim number three, the new building block of our systems will be around the context, the context meaning the data. There's going to be different perspectives to be looking at contexts and it's very much related to the old world. Nothing too new here. So right in the old world we used to have data APIs and orchestration, right?

Because we need to have different ways to look at context to really understand how it's our new building block. So we have data, APIs and orchestrations. So in the world of AI, it's context, MCP, let's say agent to agent, communication and workflows. So if we're looking into coupling, we can say that data is a tight coupling. If you're coupled by data, you're very close.

If you're coupled by APIs, you're loosely coupled. And if you have a complex flow that you're both part of, it's something very complicated. Nothing new here. Shocker. Old standouts and old concepts still can help us understand what's going on.

So if in the new world we have our team structure looking like this, in the new world we're thinking it will look like this, but in reality we'll find it something like this. So claim 3.5 is agents properly built will uncover the strong couplings between the teams because to be able to actually get the most out of the agents will we will have to go and do the hard work of the human element and the team structures and where we're curating the data to be able to expose all of this in a very neat way for our agents to perform. Okay, so a little bit more about the retrospect perception. Nevermind, never mind the world, it doesn't matter. So again, if we're going to start to look at it in a different way, not from these components, let's look at it from context.

So it can be, we can have, for systems we can either have state, which is our data, or intelligence, which is our logic. So if we go to AI, we can look at the concept that we have temporal context and we have semantic context. So basically we can take a look at it and say it's sort of like a short term, long term memory, right? Temporal is everything that's happening for me on my day to day and semantic is my long lasting memory. And if we start breaking into it into like being a little bit more technical and putting it in boxes to how we work on the Day to day and the different layers that we have in a company.

So we can say that a session is a session, right? I assume everyone uses Claude code or something, some other tool. So a session is the session that you type in personal. It's like my identity for using this agent. And then we go into the semantic things that are bigger.

We have team context, project context, organizational context. And just imagine the last time that you took on a task. Let's say it's a ticket, right? Let's say it's to fix the bug. I'll use it.

Maybe an example from my world. Let's say we have a bug in how we report analytics on rules being enforced in our AI code review. Look how much I've said, imagine how much I already know just to be able to say that this is what I need to do, right? So think of an example for yourselves, right? And this is the crisis because we're going very far, very deep, very fast.

But the reality is that all of those things they map up into tribal knowledge. That's the official word. But you know, it's very common things, right? So my personal context is pest, familiar executions, right? Or my organizational context, let's say it can be the company strategy, vision, all hands, right?

How do we give all of those things as contexts into our systems? And that's the gap, because my comment session, I know it, right? But there was a lot of actually companies that are, you know, emerging around trying to solve this gap in an optimal manner. Right? But this is something that we need to understand that it's truly a gap to be able to build those systems.

So from that world came context engineering. If someone doesn't know the concept. So basically context is the context refers to a set of tokens included when sampling from an LLM. The engineering problem is actually doing it well without getting all of the bad parts of it. So if what we need is more context, let's just throw more at it.

Well, you can't, Tom. You can't just throw more context at it because we'll have a bunch of issues. That's the prompt. Create an image shouting, add more context. I'm facepalm.

That's it. I love it. So what's the problems that we have with large contexts? A little bit. Some examples I'm not going to get really, really deep because it's a whole domain.

So if you don't know, there's a concept called lost in the middle for some reason, the LLMs are really good with the context at the beginning and the context at the end, but they lose the middle. Just like humans, right? Our attention Spanish gets lost in the end and you're like, okay, okay, what did you say? I remember the last two words. So they do the same.

We can have rot, we can have drift. We can find ourselves in a situation that because the model has conflicting data, they will give us conflicting answers depending on subtleties. There's also a concept of a needle in a haystack. If you give it too much context and one important detail, it might just lose it. It won't know that this specific detail was important enough.

And all of this ends up for us as users of these systems, we just call it hallucinations, right? It hallucinated. It doesn't know. It's not accurate, it's not true, it's not this. But in the end, it's an engineering problem that we can choose to solve if we go into this property.

So I started by saying that context is queen and it's the new building block. And this is why I'm going deeper into this. Because if you see and if you starting to get where I'm going with this is that at the end the context is the circles, right? So we have our logic, we have our agents, and we have our context and everything. Like when you draw the diagram, it will be around this and it will help us reimagine our systems at the end because it will make us look into the coupling and let's say, misorganization, mis boundaries that we already have.

So going even further, so we have the context, we understand that we need to be able to, if we agree on the fact that we need to decouple the different tasks that we want to complete for different agent, different contexts. So we need a way to be able to expose the context across different areas of our system. We have the MCP very commonly used today. It's a protocol for getting context in a dynamic manner. And this is what enables us to build agentic workflows.

Agentic agents. Okay? So if we say agentic, we mean that this agent can autonomously plan, make decisions and take actions to achieve a goal. It gets a small amount of context. It says, oh, I need this bit.

I go to this mcp. I get it. Okay, now I know that I need this bit. I go and I get it. And this what makes it agentic until it can complete the round.

Okay, again, great guy. Set with him for a chat. Actually was pretty long time ago. 4.1. If you're keeping up with the things and One last thing here on communication protocols is the agent to agent.

So if we want to have decoupling and we want to have expertise for each agent, so we need to give them a way to communicate. We need to have a concept for them to communicate with each other. There's the agent to agent protocol. To be honest, it's sort of partially adopted. This is why I added the thing internally.

For example, we don't use it. Our agents either communicates with APIs, MCPs, or even God forbid, just a method inside of the same service. No need to overcomplicate everything. God forbid your system to be simple. So it's also here.

So the context is the building block, right? And if you start pulling the context into different places, you start seeing that, okay, this block really is highly coupled and can connect together, but this block is something else and it can actually be something that is not that close. So it will help us sort of understand. And maybe with this also comes the transformation, right? If there is a group of agents that can solve a problem, maybe we don't need a team the size of 12 people to solve it, maybe three is enough, I don't know.

But these are the things that we are seeing. So it remains the building block. So avoid context hell. Okay, so my, this is the sort of summary for the bunch of stuff that I just thrown at you. Separation of concerns still a thing.

Don't just throw everything at the AI. Use the communication protocols, understand your semantic boundaries and your temporal boundaries and perform oscillation when where needed and go to a multi agent architecture if needed. And for context engineering go into that. There's so much to be said on that. We can do another I think 50, 40 minutes.

But this is just a teasel. You can read online as much as you want. Awesome. So claim number four, poly designed context boundaries and accumulation will lead to context hell. It's sort of, let's say like the pain point that at some point companies started feeling when they started building data lakes and suddenly everything was, you know, all the data was mishmashed together and the expertise was spreaded around the company, you know, so we'll reach to the same point here if we want take a step back and look at the flows and agents that we're trying to build.

Okay, so that's on the context and I've talked about the workflows a little bit. Let's give this some structure. Okay, so another transformation that we're going in for. This is like very important in how we communicate with these systems, but also how we build the systems is the fact that we're transitioning from predefined software to reason, right? So let's say a workflow is a business flow, right?

When you code, it's a flow. When a customer gets, I don't know, let's say you work in a company that builds support tools or whatever you do, everything is a business flow, right? And in the old world you would have to have an engineer, an architect, a product manager, whatever you want to call it. They have to sit down and define every if else they have in their system, otherwise it doesn't work. Anything that is not defined is considered out of scope or not supported in your system.

And everything is very, very deterministic. The change we're going to is that agents reason. So if you give them the right context, basically they can go and solve a lot of use cases, right? So you can do, they can do the orchestration, you can choose to have specialized sub agents to solve different types of problems and you can have distributed decision making. And this is inherently.

And also it's not deterministic, right? It's implicit. But this is very different from how we used to build software. It changes the product and how we use those tools. So it's important to understand this is another transformation.

So claim number five, context. When you think about them in the context of workflows, they are both organizational memory, but also their decision rationale. Because the agent is the one that's making decisions. The agent knows the ifs and elves. Okay, maybe you give it more context, maybe you give it prompts, maybe you give it a lot of things.

At the end, the agent does the ifs and else and they make the decisions. Okay, how are you with time? You excited? Yeah. Okay, that's a mid level reaction, but I'll take it.

Okay. Okay, so find the smells. Once again I want to give some examples to when to level up or when to push things from inside of Kodo. So I talked about when we were doing code generation as a problem we were solving. Now we're solving code review.

So basically what happened is that due to the same legacy, we ended up with an IDE agent that does code review and a git agent that does code review. This also, all the problems are very old world, right? We needed to do. We had the same logic defined twice. We had to support two systems and it was creating an uncohesive experience for our customers.

Right. And everything that was implemented in the git, then we need to go and implement it in the IDE shocker we just created a component that everyone can use. We exposed an API. I know, revolutionary, don't you think? Another thing is at some point in time we also reached a limitation of the single agent.

So basically we had an agentic agent that does the code review, the one in the git, the one that we ended up connecting to and exposing an API over. So we had a single agentic agent, but we wanted to give better experience for our customers and this drove us to do a whole RE architecture of our system. Right. So it ended up something roughly like this. It's not exactly, but instead of having one agent, which is like a general review agent, so now we had an agent that does context collection that can get the diff from the git and do the things.

We have an issue finding that finds bugs. We have a compliance enforcer that knows how to get the rules, the MD files, the tickets, all of this. And in the end a judge. An LLM is a judge. There's many more to these boxes, but just wanted to give you an example of a transformation we had to do.

It's very technical and actually all of this is, it's not microservices, it's all running in the same backend, like the way that we know it. Just each one of these is an agent of its own contextually. If anybody wants to read more, there is this, but on your own time. Okay, so find the smell cheat sheet. So I talked a lot about the context and it being the building block and what are the tails that are getting us there, right?

So for example, for example, do we see accuracy or performance dropping over time? Right. Hopefully you have evaluations over the tools that you have in your systems or build. So this can be, for example, a context drift, right? It can come from this.

So if you took a picture, great. If you want to talk about it afterwards, more, great. I'm going to carry on because I have 10 minutes and I still want to get to the carbon based life forms. Right. I was in a conference last week leading a roundtable and this one guy, he would not say humans.

He was like the carbon based life forms, the non carbon based. And I'm like, this is what we came to. I mean it was a roundtable about AI, obviously, but still. So this is us, the carbon based. So claim number five, the systems are changing and we have to be flexible.

I said in the beginning, I can't predict the future. But what's true is that the thing that is true for sure is that things are changing. Where will it Go, what will happen? But we need to be like this. I think that something that people don't talk about enough is how we feel.

This is like, it's a very, like it's crazy times, right? Usually as it people, technologists, whatever, we're used to automating things, right? Just in other sectors, right. Not ourselves, right. Suddenly someone you know, they're saying the code generation problem is solved.

I mean, we can argue here and there, but they say it's solved, you know, and it's automation on us. So we have feelings about this, you know, so people are worried about job replacement. And I feel a lot about burnout from being overwhelmed from all the tools and the changes and keeping up with things. People have trust issues with AI, but you know, everyone are telling them, use AI, you have to use AI. And believe me, your managers are telling you you have to use AI because the board is telling them they have to use AI.

So they tell you you have to use AI. And maybe you, as managers tell your employees they have to use AI. And it's like a chain and when you have to do things, it kind of ruins the trust. And there's shadow AI. People are using things.

Even like sometimes companies are not trusting in the AI, but employees want to try. But tomorrow, tomorrow could be a better day. It doesn't have to be the bad apocalypse. I don't know if it's the good apocalypse. I don't think it's going to be an apocalypse anyway.

No apocalypse. So new jobs will appear, changes potentially will plateau. I don't know. People are talking about AGI last year they said 2026. I'm still waiting.

I don't know. Trust will build more companies will adopt AI and understand how to do this well. And with guardrails. The scope of our work will change. I hear about, I don't know, crazy structural transformations in different companies.

I don't know. Suddenly, like, I don't know, drop everything. Everybody's a builder now. Teams are the size of two and that's it. That's what you do.

I don't know. Companies are experimenting. Structures is going to change. Sometimes it will be good, sometimes it will be not as good. But the scope of our work and the expectations of us is going to change.

How we learn is changing and has to change. We need to be able to be faster in the pace of how we learn. If you are too tired to learn, it will be over time, very hard. This was, by the way, also true in the old world. Old world, like two years ago.

Not that old. You needed to always keep up with technology. The thing is that now everything is moving so fast and there's such a FOMO hype type of situation that to be able to keep up, you need to adjust yourself with the skills of learning. Our jobs will change. Some jobs are disappearing.

They're talking a lot about middle management as something that might less have space in the new world. Low level support roles, stuff like this. Personally, I think that the junior engineers will make a comeback. I don't know when, but that's like my belief that once we get settled in with this. But we have a lot of things emerging, right?

So two years ago nobody knew what a forward deployed engineer. If you don't know what this is, it's like mega solution architect that is assigned to a specific company and goes and works with them on a daily basis. This is like in 2026 the number of open positions for deployers engineers has increased by 1000%. You know, so there's AI leads and agent creators and AI supervisors. Like who would have thought, like what you do in your job is to supervise an AI.

But that's the reality of things. I think this one is very important as a concept. We need to understand that how we interact with system. I touched about this a little bit. Is also changing.

So it's going to change from command to intent, right? It's not write this line of code is I want to solve a bug that solves the pain for this customer or or maybe I want to build an app that helps people do this and that. And we're also going to move from sequential to parallel. We're used to working more on, let's say maybe one or two tasks at the same time. Now it's going to be a lot more because we're going to be empowered by AI.

This has its downfalls as well. It's also contributing to burnout. But it's something worth putting on the table and it's going to move from micromanagement to oversight. We will build trust in these tools and we will be able to give them more leeway to make decisions of their own. And it's going there.

So one last note before I do my summary. I added this slide yesterday because at the conference last week and I know about the concept and I know that also we have customers that are doing the dark factories. So the concept of dark factories is from when automation has, you know, moved into the physical world and they're starting to have factories, actual factories that are with robots instead of humans. And then you don't need the lights on. So it's a dark factory.

So it's happening in software now. They're building entire teams that have no humans in it. So the dog. So in the conference we had like several people come to our booth and be like, okay, so do you support this use case? We do, but I mean it's really becoming a thing.

So the oversight paradox, basically it's a very. Just uncomfortable, right, to think that we're going to have a whole team and it's going to be autonomically solving everything. A product, part of a product that is only by oversight. It's going to be redefining engineering as a whole. Right, because the craft won't be coding.

And at the end our guardrails and our gates for code quality and governance is going to be like a real problem, a real blocker to be able to move to this. Will this really take over the world? I don't know. As I said, I don't know the future. But I know that a lot of companies also big enterprise customer, not customers, companies experiment with this.

So be flexible. So to put it all together, I have defend, evolve, redesign. So defend good boundaries, good architectural understanding from the old world. It's not going anywhere, anywhere. At least in my eyes.

It's the same old thing. 20 years of experience doesn't go to the trash that easily. Redesign. Think about your organizational impact on agents. Think of suboptimal communication patterns, your team structures, context, coupling everything that I talked on and evolve skills.

Your tools fill the gap of the context. Redefine the human versus agent roles and responsibilities as it's becoming a new actor in your socio technical. You see how I closed the loop in your socio technical system and think about roles in general. As I said, principal engineer to product manager. Yeah.

Okay, so map your smells. That's me. If anybody wants to follow me on LinkedIn or wants to talk afterwards without I don't know if you don't catch me and you want to. So I'm there, I'm very active and that's it. Thank you very much.
