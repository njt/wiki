---
url: https://rubriclabs.com/blog/always-on-agents
date_fetched: 2026-11-09
---

Before ChatGPT was released, I used to imagine AI would be something like Jarvis from Iron Man. I imagined it would always be running, it would have a personality, and it would have the same awareness of my environment that I do.

But today's chatbots and agents aren't like this. We spin them up for tasks and projects, but if we don't send them a message, nothing happens. They're dry and robotic, and they only know the context we give them.

I have a hunch that the reason people disagree about whether we've reached AGI is **more about form factor than it is about model capabilities**.

To (over-)simplify, I would define the always-on, Jarvis-like form factor as:

- single-threaded
- proactive
- situationally aware

Whereas chatbots and agents are:

- ephemeral (short-lived)
- reactive (they only work when explicitly started)
- context-dependent (heavily influenced by whatever happens to be in front of them)

In this post, we'll talk about how we're designing architectures for always-on agents, and the limitations of today's models that make this hard.

## Single-threaded

Instead of managing a bunch of different threads, and setting each one up with the right context for our work, we just want to talk to **one "AI" that knows us and handles everything**.

Let's say we woke up sick. We want one agent to be able to cancel our meetings, order soup, look into our symptoms, and book a doctor's appointment if necessary.

At first, this sounds simple: just unify everything into one thread and call that our single-threaded agent. The hard part is context window management, since passing everything through a single agent will quickly fill up its context window.

We also often want to do several things at once. Booking the doctor's appointment or ordering food might take a while, because they're heavier browser-use tasks, and we want to keep chatting about our symptoms without waiting for everything else to get done.

### Compaction

Compaction is the de facto standard for managing context windows, but we've found that **compaction lobotomizes the agent's momentum**, especially when it's done by a third-party process.

It makes more sense to give the agent tools to compact and awareness of how full its context window is, and then just **let it manage its own tokens**, so it can keep its momentum while avoiding overflow.

Tangibly, we give our agents:

- awareness of their token usage at every turn
- a `foldContext`tool, which replaces a range of messages with a summary the agent writes
- a `dropContext`tool, which removes a range of messages from context

In practice, our agents tend to wait for a lull, like the end of a task, before they compact. Then they fold away the reasoning and tool calls they won't need again, and keep everything that's still in play.

### Dispatch

Even with good compaction, one thread fills up fast if the agent does all the heavy work in it. So we also give the main agent **sub-agents it can dispatch work to**.

This kills two birds with one stone: it frees up the main thread to hold more, higher-level context about everything, and it lets the agent multitask on deeper work, like the food order and the doctor's appointment, while it keeps talking to you.

## Proactive

Proactive agents can **do work without being prompted**, and they can send unsolicited messages. For example, if the agent knows you're flying to Austin on Wednesday, it can find the check-in details for your Airbnb, work out the best way there from the airport, and send it all to you as you land.

To unlock this behaviour architecturally, we first need to get away from the request-based nature of the agent. Typical systems act a lot like synchronous functions: we send a request and wait for the agent to return.

For proactivity, the agent needs to be **awake whenever it wants**, and to message us regardless of whether we sent it a request.

### Heartbeat

We give the agent a heartbeat. A heartbeat is just a simple message, "heartbeat", that we push into the chat history. **It wakes the agent if it's asleep.**

We can run the heartbeat on a cadence (for example, every hour), or have the agent manage its own awakeness by scheduling its own heartbeats.

### Messaging as a tool

We also give the agent a **tool for sending messages** to the user. In a normal chat, the agent's last message in a turn is its reply to you. Here, it only reaches you when it calls the tool, so it can wake up, do some work, and decide there's nothing worth saying. Its last message stays in the thread as a note to itself about what happened.

In the Austin example, the agent wakes up on a routine heartbeat and discovers the flight. It dispatches a sub-agent to pull together the directions and check-in details, then schedules a heartbeat for around the time the flight lands, so it can message you at the right moment.

The Grid

What we’re learning about building with AI. Two short emails a month.

## Situational awareness

The prerequisite to proactivity is **knowing what you're doing, and when**. We want our agents to be able to act on incomplete information from us, and, like in the Austin example, even start working on triggers from the world, not just on direct instructions from us.

### Broad access

At a basic level, we can enable situational awareness with broad access: give the agent **full access to our email, browser and file system** so it can discover flights and accommodations we booked, search past orders, and find patterns in our behaviour that surface our preferences.

The one simple trick here is just giving agents computer use tooling and access to our computers. Coding agents got great with one simple tool for running terminal commands on our computers, and general-purpose agents can do the same with computer use.

But agents don't know what they don't know, and being technically capable of discovering context often isn't enough to have a real-time understanding of what's going on.

### Notifications

So we give agents a notifications system, where **information is injected into the agent's context window as soon as it's available**. By subscribing to events in the world, agents can discover important context sooner and step in when they're needed.

For example, if your flight to Austin is cancelled, the agent can immediately start looking for alternative flights while messaging you to coordinate, instead of waking up an hour after the problem is solved to see what happened.

To implement notifications, we give the agent an inbox, and documentation on how to adapt webhooks and polling jobs to push events into that inbox.

One problem quickly emerges: **the inbox fills up with noise**. Spam offers in email, Slack messages that aren't addressed to the agent, GitHub events from repos that aren't in focus. If we wake the agent for every notification it receives, we could be doing so every minute, burning tokens and clouding its context window.

One way to cut the noise is to classify notifications as important or unimportant with a smaller model, such as a system one model like Jev. **Important notifications wake the agent**, and unimportant ones pile up so the agent can read them in bulk when it wants to.

## Discernment

Architecturally, everything above is a fairly simple series of interventions. Make the agent single-threaded, and give it tooling for compaction and dispatch. Give it a heartbeat, and separate when it's awake from when it messages you. Give it broad access and notifications so it can discover what it needs to be helpful on its own.

But for the models available to us today, this re-architecture is a paradigm shift in expectations. Before, we would give our agents a task and the context to complete it, and they would work until they finished and returned to us. Now, we're giving them a raw, unfiltered stream of noise and asking them to **figure out what to do**, and how to do it, while they keep processing more noise.

Everything in their training (specifically their RL arenas) has taught them how to crush tasks. So these agents demo really well when the signal to kick off work is very clear, but they struggle to figure out how to be helpful in ambiguous environments.

For example, agents processing messages on Slack often decide to act on messages that aren't addressed to them. Then, after discovering they lack the mandate, access and context to complete the task, they settle on writing a design doc and sharing it. This is useless and embarrassing.

On the flip side, agents often miss real triggers to do work.

Agents are either **too eager to do busy work or missing opportunities to be helpful**. Calibrating them to the right balance is more art than science.

### Designing for discernment

The behaviour you want to design is for the agent to **start out shy, and over time build the confidence** to act proactively as it builds itself a lane where it can make an impact.

This hinges on good system prompting and good use of memory. Concretely, we want:

- a system prompt that's really clear about what the agent is responsible for, and that doing nothing is totally fine
- the agent to ask when it isn't sure whether something is its job, instead of guessing
- a memory of how we reacted when it did act (did we thank it, correct it, or ignore it?), which it checks before it acts again

I also think there are huge opportunities in **new RL environments that reward discernment** and memory use for this new paradigm.

## Back to Jarvis

There's still a long way to go before agents feel like Jarvis. Some of the gap is architecture we already know how to build: one thread, a heartbeat, and an inbox. Discernment is the hard part, and it's the part I'm most excited to work on.

We're building always-on agents at Rubric right now. If you are too, we'd love to compare notes.
