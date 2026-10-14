---
url: https://everything.intellectronica.net/p/five-mistakes-people-make-with-agentic
date_fetched: 2026-10-14
---

I can ask an agent to do something, watch it produce good work, and still spend the next hour telling it what to do next. The agent is busy. So am I. Somehow, I’m still deciding every next step, with another conversation to attend to.

Moving into **factory mode** changes that relationship. I give the agent an outcome, the context it needs, and a way to check its work. It carries the task through, including dealing with problems along the way, and comes back with a result I can inspect.

I gave a **Factory Life** talk about five mistakes that get in the way of that shift. I’ve seen these patterns in my own factories, in my work with larger teams, and in conversations with other people building theirs.

Hugo Bowne-Anderson and I are teaching

Build Your Agentic Software Factory. If you’d like to build your own, there’s more about the course at the end.

## What I Mean by a Factory

A factory is a repeatable way to get work done through agents. You provide direction; the agents have tools, context, and enough room to make decisions. They can work asynchronously in the background, while the surrounding system starts jobs and brings results or decisions back to you.

The work might be software, research, publishing, operations, or a mixture. I still decide what matters, what the agents are allowed to do, and whether the result is good enough. A failed check can send the work back for another pass. What we learn should improve the next run.

These choices depend on the task. Sometimes I want an interactive conversation, the strongest model earns its cost, or several agents are exactly what the work needs. The mistakes start when I make those choices by habit.

## 1. Micromanaging the Agent

This is an easy mistake to make because it feels responsible. I want the result to be right, so I stay close and keep supplying the next instruction.

There are two versions. In the first, I fall back into chat mode: the agent finishes a step, I give it another, and the plan stays in my head. In the second, I write such detailed instructions that the agent has little freedom to adapt when it learns something new.

I once put an “always write tests” rule in my standing agent instructions. When asked to change a README, an agent tried to write a test for the README. The agent followed the rule with rather more enthusiasm than the task called for.

Exploring a problem together is a good use of chat. But when I mean to delegate, the agent needs enough direction to complete a stretch of work without my constant input. If I keep wanting to intervene, it’s worth checking whether the brief leaves an important decision unresolved.

A rich specification can give the agent plenty to work with. What should exist afterwards? What should it read? What must it preserve? Which decisions can it make? How will we know it worked?

For example, if I want a research brief to help decide what to teach, I can name the decision, relevant sources, how to handle uncertainty, and the requirement to return an unpublished draft. The agent can choose the search route and organise what it finds. I don’t need to choose every query or prescribe the paragraph count.

Hugo and I have talked about a good practical test: could I hand over this task and come back an hour later to the result I meant? That’s the level of delegation I’m aiming for. I still want checkpoints when a decision changes the outcome, cost, audience, or risk.

## 2. Using the Best and Most Expensive Model for Everything

I like capable models. When I’m exploring an idea or trying to understand something difficult, I often want the strongest one available to me. It’s tempting to make that the default for every job in the factory.

Background jobs keep running when nobody is watching the cost. They can use up a subscription allowance as well as a cash budget.

Choosing a model also means choosing its **reasoning effort**: the setting for how much work it puts into thinking through a task. I also consider how long the job can take and how good the result needs to be. A fast response matters when I’m working interactively. A background task may have room to think longer, provided it still meets its deadline.

The useful comparison is what it costs to get a result I can accept. A cheaper attempt that needs repeated repairs may cost more overall. A larger model can earn its price by resolving a difficult question or catching a mistake before it spreads.

I choose by function and task. Brainstorming, analysis, planning, and review can benefit from deeper judgement. A well-defined task with a clear specification and checks is a good place to try a cheaper model. Difficult execution may still need the stronger one.

Benchmarks help me find candidates. Representative jobs from my own factory tell me whether those candidates fit. I compare the results against the same bar, including elapsed time, retries, my corrections, and spend or quota. If the cheaper choice keeps failing the check, I escalate.

## 3. Giving the Agent No Guidance, Taste, or Way to Check Its Work

An agent can read a repository or a collection of documents and still have no idea what good work looks like here. It doesn’t automatically know our settled decisions, preferred vocabulary, taste, or the exception that matters most.

The agent will usually try to do something anyway. It fills the gaps with patterns it has learnt elsewhere and whatever context happens to be available. The result can be polished and coherent while missing the point of the project.

General competence only gets us so far. The knowledge that makes work fit a particular person or organisation often lives in their head, past conversations, and examples nobody has collected.

I try to give that context a home:

- **Standing instructions**such as- `AGENTS.md`hold decisions, constraints, and conventions that apply across tasks.
- **References and examples**provide facts and show what I like, ideally with an explanation of why an example works.
- **Skills and supporting tools**give the agent methods it can reuse, along with scripts, tests, and access to the relevant systems.

The current brief still needs to describe the task, and a worker on another machine needs access to the relevant instructions and references too.

More context isn’t automatically better. I want the relevant material to be easy to discover, without burying it in a manual full of things that don’t matter to this job. In Context Camp, I described how I organise and retrieve knowledge for my personal agents.

When the same mistake keeps happening, I ask where the process failed. Was the guidance missing, undiscovered, ignored, or contradicted? Did the agent have a way to recognise the problem? Adding another emphatic sentence to the instructions is only one possible response.

## 4. The Puppet Show

You start with a CEO agent, then add a strategist, a writer, an editor and a critic. Before long, you have an impressive organisational chart and a fair amount of conversation about the work.

I call this the puppet show: inventing a cast of personas before working out which functions need to be separated.

I used to dismiss multi-agent systems as anthropomorphising. I’ve changed my mind, and I now use multiple agents a lot. Separate workers can be very helpful when they handle distinct parts of the work.

Independent source gathering can run in parallel, and a reviewer can check the requirements and evidence. Some workers need different tools or narrower permissions. A background monitor needs to keep running over time, while a reviewer may be brought in for a single task.

Each extra agent needs its own context, a handoff, and a way to combine the results. Closely related tasks can become slower and less coherent when split unnecessarily.

I start with a capable agent and add functions as the need appears. Before adding another worker, I ask whether a better brief, more context, or different tools would let the existing agent do the job.

When I do split the work, I want to know what each agent receives, what it can do, what it returns, and how we’ll check the result. I also want to see whether the split helped. If it only adds handoffs, I can remove it. There is no prize for employing the most imaginary executives.

## 5. Treating Rules, Skills, and Prompts as Deterministic Infrastructure

The guidance in mistake three helps an agent make better decisions. Some limits also need to be enforced by the system around it.

Imagine a drafting agent with a publishing tool and credentials. Its instructions say “do not publish without approval”. The agent still has the capability to publish. We have asked a sentence to enforce a boundary.

Models interpret instructions. Their behaviour can vary with the model version, context, tools, and material they retrieve. I’ve seen odd behaviour after changing models and then realised that old instructions or skills needed another look. A page or email can also contain instructions that the agent should treat as untrusted material.

A skill can include a script that performs an exact check. That helps, but we still need to ask whether the agent can skip the script or continue after it fails. If it can, the check is optional in practice.

For things that must be enforced, I look at the system around the agent. File permissions and a sandbox can restrict which files it can reach. A drafting identity can save work while lacking permission to publish it. A required check can run before an action and block that action on failure.

The whole route needs checking. Hiding a Publish button does little if the agent can use the same credentials through another tool. A control can also be misconfigured. I want to check both that the worker can do what it’s allowed to do and that a forbidden action is blocked, using the tools it will actually work with.

There are two separate questions here: did the work pass the check, and is the agent authorised to take the next action? A correct draft doesn’t acquire permission to publish itself. Keeping those two questions separate lets the agent get on with the work while I retain control of what it’s allowed to do.

## Five Questions for Your Next Task

Take a task that needed more supervision than you expected. Where did you keep intervening?

- Am I delegating a result or directing every step? 
- Does the model fit this part of the work? 
- Can the agent find our quality bar and check the result? 
- Does each agent have a distinct function? 
- What enforces the boundary if the agent gets it wrong? 

Change that part of the process and try again with work you can inspect. Stay close at first, then step back as the results and checks become dependable. That’s how I want my factories to grow: through experience with the work they’re doing.

## Build Your Agentic Software Factory with Hugo and Me

These are some of the decisions Hugo Bowne-Anderson and I will help you work through in **Build Your Agentic Software Factory**, our two-week course on Maven.

You’ll set up a factory, build a small tool you can use through conversation, extend that approach into a working web application, and learn how to check, publish, and share what your agents build. Bring an idea from your work or personal life, or use one of the starter projects. The aim is to leave with something you can use and a process you can repeat for the next idea.

We’ll work on writing specifications, giving agents context, deciding what to delegate, and inspecting the results. There are two live workshops, weekly office hours and project feedback, recordings, course materials, and a community of people building alongside you. Every student also gets **$500 in Modal credits** for running agent jobs or hosting what they build in the cloud.

You don’t need to know how to code. You do need curiosity, judgement, and a willingness to experiment and check the results. Experienced developers are welcome too; there’s plenty to learn about giving agents a larger share of the work.
