---
url: https://gist.github.com/njt/b678d6a9967bc3dffe43dff825db5967
date_fetched: 2026-08-21
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Key points — The architecture is born from a single insight: selectivity, not automation, is the real problem

The goal isn’t to auto-review every line; it’s to reclaim senior reviewer attention by automating the mechanical part and surfacing only findings worth human judgment. The system must be selective, not maximal. This principle drives every design choice: multi-agent fan-out, confidence scoring, human-in-the-loop gating, and retrieval grounding.

Pithy and provocative quotes

“The problem is not to have an agent that can review your code automatically … but instead the problem is the selectivity — which findings does the agent take and which ones does a human judgment actually get spent on.”

“Hallucination is a feature, not a bug.” (Said while designing against it with citations and confidence.)

“If you have generated one token, tell me what was the output of that. If you cannot justify the output, I mean, dude, fuck you out.”

“I don’t like Whisper. … Am I going to pay you even a single penny to dictate? Are you serious? The whole model is flawed, man.”

“The system optimizes for surfacing findings worth a senior attention and deferring the rest — not for maximal output. Selectivity is the seed of this whole architecture.”

“A review agent is not a replacement for human judgment. … Human judgment is very scarce. Can we put up this particular situation so that human only focuses on where it actually requires?”

“If you remove a component … you will see a new problem coming out of it. … Necessity is the mother of invention — we’ll create that necessity intentionally.”

Tools, practices, and methodologies

Selectivity-first design – Frame the problem as “reclaim senior attention,” not “automate review.” Every architectural decision flows from this.

Map the mess – Before any tech, document exactly what a human reviewer does today: micro-decisions, knowledge recalled, fatigue points, failure modes.

Trigger and output shape – Define a precise trigger (GitHub pull_request webhook) and a structured output: a list of findings, each with agent_type, severity, file/line, confidence, and rationale.

Fan-out/fan-in multi-agent pattern – Four specialist agents (security, quality, testing, docs) run in parallel, each grounded by retrieval; an aggregator merges, deduplicates, and scores.

Retrieval-augmented grounding – For every PR diff, fetch only the most relevant code slices (semantic memory via vector search), past reviews (episodic memory), and conventions (procedural memory) to stuff into the specialist prompts.

TigerCloud as single data store – A managed Postgres with pgvector (vector search), pgvectorscale (DiskANN index for large-scale embeddings), TimescaleDB hyper tables (time-series events), and continuous aggregates (precomputed cost/latency summaries). Avoids three separate databases.

Redis + ARQ for job queue – Webhook ingress acknowledges GitHub immediately, enqueues job to Redis (via ARQ), decoupling fast ack from slow LLM work.

LangGraph orchestrator – Defines workflow as a directed graph; fan-out via Send API; checkpoints to Redis for resume after failure. Abstracted behind a WorkflowEngine interface to allow swapping to Temporal later.

Confidence gate and human-in-the-loop – Findings below a confidence threshold or marked critical are routed to a human approval queue; above threshold auto-post. Maturity raises the threshold over time.

Event spine for observability – Every LLM call, tool call, and decision is recorded as an immutable time-ordered event in a hyper table. Powers trace viewer, audit trail, and cost dashboard.

Genesis Kit (AI-native engineering harness) – A loop that prompts itself: a locked DONE.html spec, context graph with invariants, five gates (G0–L4), build/debug/research loops, and an independent verifier agent that quizzes the developer to ensure understanding.

Modular monolith with separate worker pool – Webhook receiver is a stateless FastAPI service; orchestrator runs as a separate ARQ worker, both sharing Redis.

Failure mode matrix – For each component, list what can go wrong (engineering direction: timeouts, retries, idempotency; LLM direction: hallucination, drift, tool failure) and design defenses (retries, circuit breakers, dedup, confidence thresholds).

Invariants as non‑negotiables – Examples: every webhook payload must pass HMAC signature verification; every specialist finding must carry confidence and rationale; agent events table is append‑only and immutable.

Unanswered questions and omissions

Specialist agent quality and conflict resolution – The talk assumes four agents produce structured findings, but how are conflicting findings (e.g., security vs. quality) resolved? The aggregator “merges and deduplicates,” but no concrete conflict logic is given.

Retrieval design details – Chunking strategy, embedding model selection, hybrid search (dense + sparse) tuning, and how to handle very large monorepos are only sketched. The “context engineering” is mentioned but not implemented.

Human-in-the-loop queue management – Capacity planning, prioritization, and escalation rate monitoring are named as problems but not solved. What happens when the approval queue grows faster than humans can clear it?

Evaluation and regression testing – A “golden dataset” and regression gate are mentioned, but no methodology for creating or maintaining them is provided. How do you know the system is improving?

Feedback loop poisoning and model drift – The talk acknowledges these risks but offers only high‑level mitigations (“minimum evidence threshold,” “periodic prompt updates”). No concrete mechanism for detecting drift or safely incorporating disputed feedback is built.

Security of the agent itself – Prompt injection, adversarial code in PRs, and model output sanitization are noted as trust boundaries but not deeply explored. How does the system prevent an attacker from manipulating the review?

Cost optimization and token economics – The speaker mentions a future session on token optimization; the current design does not address caching, model routing based on cost, or dynamic budget enforcement beyond a simple aggregate check.

Multi‑repository and organizational scale – The architecture assumes a single repository. How would it handle hundreds of repos, different review policies per team, or integration with CI/CD pipelines beyond GitHub?

Genesis Kit’s inner workings and limitations – Presented as a magic harness, but its dependency on specific folder structures, manual checkpointing, and the “quiz me” protocol may not scale to large teams or complex projects. The talk doesn’t discuss when Genesis itself breaks.

Real‑world deployment and operations – Railway is mentioned for deployment, but there’s no discussion of secrets management, monitoring alerting, logging aggregation, or disaster recovery for the TigerCloud database.

Learn how to build a production ready AI agent system designed to review code pull requests. You will explore a sophisticated multi agent architecture modeled after the selective human judgment of a senior reviewer. IU Singh will show you how to develop the system component by component using an AI native engineering loop that addresses potential engineering failures, integrates robust verification gates and leaves you with a reliable baseline to extend further. Okay, today we are going to build an agent that reviews your pull requests. And here's the plan.

What we will do. We will take the difference of your code which you have added or deleted. We'll give it to maybe OpenAI or anthropic API key with a completion prompt and in the prompt we will write okay, tell me what's wrong with this code and hopefully expect some good comments out of that LLM. And okay, we can do something else as well. We, we can also add rag, we can pull some relevant files from the repository, we can stuff them into the nice little prompt and then we can call that production ready.

Right? If that's what you came here for. Please close this video. I'm seriously, please close this video. There are 400 tutorials that do exactly like that and none of them is the production ready agentic system.

Here's what we will do instead. And frankly speaking, it does not start with tech. It starts with a real senior reviewer who has 20 PRs to review today. And the quality on the first PR is not equal to the quality on the 20th one. And you might imagine some reason why.

It might be fatigue, it might be inconsistency, it might be some human judgment, it might be some miss things in the 20th PR or it can be anything which relates to a human. And I want you to think about it. The problem is not to have an agent that can review your code automatically or in automated way, but instead the problem is the selectivity which is which findings does the agent takes and which ones does a human judgment actually get spent on. Now that one sentence of selectivity is going to be the whole architecture and everything will fall around it. Here's the part which I'll also care about is I'm not going to give you a finished game or finished diagram of your architecture.

This is how we're going to build it. I'm going to show you how I finish thought about this process and frankly speaking, it's kind of a loop that runs once per component. What we do is first we'll find a human system that has already solved it, which means watching how a senior actually reads a difference of the code which you added or deleted. Then we'll bring the code based context which a senior reviewer actually thought about building in that particular moment. Then what we will do, we will reason across separate concerns.

For example, a reviewer might think about security, maybe they think about quality, they think about correctness, they think about testing, they think about documentation. Every one of them is coming from a different mindset and frankly speaking, they are very skeptical. They are providing evidence of the everything which they are talking about so that if at any point of time they get wrong, they can get back to it. And if you read those four statements from the engineering point of view, the code based context means retriever. Separate concerns means it's not just one reasoner, it has multi agent architecture.

Every site which is having evidence, which means every finding which my agent will make must carry a rational and a confidence. And the four agents which we'll design will not come from okay, let's design for agents. But actually thinking from what eventually Howard human reviews and then we will take one component of entire architectures one by one and then we'll ask a single question to every component is what can go wrong and what we will do. We'll think from two different perspectives, two different directions. The first one is the engineering direction where it all can fail.

For example, let's say you're working on the component of GitHub, which is GitHub retries the same delivery all the time. Maybe the security problem that somebody else is sending your request of PR, which is not GitHub officially. And then you have just 10 seconds to acknowledge to GitHub but your agent takes 90 seconds. All of that is engineering direction failure modes. In the LLM direction we have the, let's say the model hallucinates the retrieval pulls the wrong slices every single thing which our LLMs can go wrong.

And then what we will do for every component, we'll put those failures into two by two matrix. For example, what you know you don't know, what you'll recognize eyes when you see it and what you have never ever considered. And we'll do that and the design of the component will start to write itself and then we'll move to the next one. For example orchestrator, retriever, events, gates, five components. We'll run the same loop five times and by the end we'll be having an architecture which is fault tolerance.

We have designed already till there and only after designing the whole system we will code and frankly speaking, we'll not wipe code it like other people. We'll run our genesis ritual Genesis is in harnesses which actually give you an ability to maintain a state and to be able to consciously work with AI, to be able to develop your project or to be able to implement your architecture. For example, a context graph with your invariance. How does a definition of a done which looks like what each milestone will eventually contain and exactly what will the output of each milestones. And then the loop will drive the agent against that spine.

We'll be having five different gates to build that particular loop and an independent verifier that actually verifies whether it has done the work or not. Along with it, we'll integrate systems where it literally asks questions whether you have understood whatever it has written or not. And we'll truly show you how AI native engineering actually works when you actually be mindful about what your AI is doing. And by that point you're not depending upon your agent to think and code. You're actually directing your agent and still having the full harness of how you're controlling the agent and what your agent is doing.

You'll ship a real system, you'll have a multi agents running in parallel. You'll be having a human approval queue, a full trace view viewer and a cost dashboard that will actually going to show you the economics of your LLM. But more than that, you will have the loop and the loop moves. We'll point it at a correct previewer, an incident agent and anything of that kind. And more than that, you'll also learn how to natively work with AI and still be able to understand what your AI is doing at every single point of time.

So there's a roadmap in the description. So I'll end this entire course not by building the entire system, but giving you a baseline to build on top of it so that you can actually extend this project and natively work with it. So let's get started with our system design. All right guys, so let's get started with our actually designing the system. So first of all, before we even get towards telling you about, hey, this is how to think about, this is how to write the design.

The idea of this project is to of course tell you that okay, this is how you can design an AIPR review agent. But more importantly, I want you to be able to extend this to other projects problem, I want you to be able to extend this to other ideas by your own. And that's why I first of all, first of all want to share you that this is the lens which is behind the way we think about, you know, this Particular whole system EIPR agent, which you can further generalize to any system which you're working on now. So we usually have this template and which I have created and the first template and the first move is mapping the mess. So whenever we will go ahead and when we design the system as well, we want to document what actually happens today.

I mean if you. I'm designing an agent which reviews a pr, I want to see that what if that particular AI agent is not there? What is happening right now? What is happening without that particular particular solution, without that particular AI agent. Now let's assume why I do this, because as I say that if you remove a component from it, if you remove that, okay, you want to review a pr.

So instead of an AI which is reviewing the pr, what you're doing, you're having a human to review it. That's what you're doing right now. Given that, we assume that there is no other AIPR review agents in the world. So if you remove that component, what is happening, what you want to do, you want to observe everything which a person is going to do, you want to write those micro decisions which a particular person is going to do. It can be literally anything.

It can be, okay, these micro decisions, for example, let's say, I will take a very simple example in this case, let's say we want to build this agent how a particular senior reviewer will go ahead, will look at what code the micro actions, he will take the knowledge base, he will recall the knowledge base, he will see the architecture, decision records, he will see the documentation, he will see etcetera, etcetera. So what is he doing at this particular point of time to review a particular pr and then you write every single minute steps of that particular person doing it. Because when you do that, and then you will figure out, okay, this is the mess which we need to organize in terms of agentic design. And that's the first step. There is no concrete step, and I always tell listed to people is AI system design is just that there's a manual work going on.

You need to look at every possible steps which a human is doing, even unconsciously. And then you pick that up and then what do you say? Okay, fair enough. Now what I'm going to do, I'm going to go ahead and figure out which is mechanical, where human is doing his judgment and where we can, you know, put LLMs, where we can put deterministic ML, where we can put anything. So we need to figure out that this is what we're going to Do.

So the first step, which we're going to do in this project is figuring out what happens today, right? And more importantly, I want to understand, okay, what triggers it. Basically, if my ifs. If what triggers a senior engineer or some, some engineer to review your PR, it's like, okay, as soon as the PR is made, he will get some notifications on his GitHub or probably Discord Alert or Slack Alert, or probably some person is going to say, hey, I made a pr, can you review it? And then he will make a comment.

And then, so we want to write all those minor decisions and we're going to do that and we'll explicitly say, okay, this is where the human is required and this is the mechanical stuff is happening. And we will also figure out where it breaks again. I'm not yet on AI, I'm just on humans reviewing the PR and figuring out what it does, how it does, why it does, and where human fails. That's the first step. Second move, I want to make it more explicit to talk about what is going to be my trigger and what is going to be my output.

And more importantly, I want to just talk about one thing, is if a person, if a particular person wants to review a particular pr, what happens? He gets a Slack notification, he get a GitHub notification, he gets an email notification, he gets some person saying, hey, can you review my priority pr? Can you, can you, you know, tell me that, okay, I have made a pr. Can you please, please go ahead and review it? That's, that's what the trigger which I want to give to my agent is.

I want my agent to be triggering its workflow at a very precise trigger. And it should be a very precise. For example, the way I mean by precise is I want to take one example. Let's say that you're building some insurance approval agent which approves whether to give the insurance to this person or not, right? What happens is any email which comes in, what will trigger the agent, we don't want that claims comes in.

That's not a, that's a very vague trigger. We want a very precise trigger. So in that case, it would be a claim which arrives at this particular email ID and has a word known, has a keyword such as claims. Similarly in this, we want that, okay, we want, anytime a PR gets posted, we should be able to create some GitHub webhook and use the GitHub webhook stuff to be able to get this on our Slack. And that particular PR by that particular person probably can be One of our trigger.

Now it's not just about having a trigger. We want to output for example into the our insurance claim example. We want to be able to say okay, it should be able to produce some output, right? Even in this what it will do, it will say okay code. Eventually a structured PR review is posted.

The way a human would do is a structured PR will eventually get posted. Now let's assume like there are several types. A person will eventually think about a particular PR or maybe any GitHub PR which eventually gets made. The first point is that it should be able to think in terms of okay, because my PR is posted, I should be able to test across the verticals whether it is readable, whether it follows a security principle, whether it is, you know, that the test cases has been added, whether it, whether it actually confirms and works based on architectural style. There can be multiple parameters on this particular engineer might be testing the pr, might be reviewing the PR and then he will make a comment and then the person will come, the one who makes the code and then he will come to this engineer and say that hey, this is.

And clarify if something was not clarified on an engineer or fix it and then go send it sends it one again. Now there can be also possible cases where an engineer needs to even revisit some other people, right? I mean, who is more senior to him so that he can understand if he's not able to understand anything or if he's conscious about anything, right? So he should be able to go ahead and, and post that. And if it is an open source, let's say that if it is a PR of a big organization, they have, they have a lot of employees and is an open source GitHub, right?

If a particular engineer finds a very big security vulnerability, it should not be able to go and say hey, this is what is going on, right? It should not be able to literally point out the security vulnerability right away. It should eventually go or raise something behind the scenes basically not in the public, but more importantly raised to the directly by via Slack chat or something. We don't want to talk about what is a security issues publicly into an open pr. People can actually misuse it.

So all of that eventually comes down to move number two and move number three is figuring out at which point of time I've told you multiple steps. What A senior engineer will look at the PR and then he will look at the code differences. Then he will look at its four to five parameters which he wants to test a particular PR based on whatever is the Guidelines of testing a pr, right? There can be multiple steps which is involved and we'll talk about those multiple steps in a bit. But we want to assign each step a component.

A component can be our trigger, which means deciding that okay, this particular component is a trigger. Now if you want to fetch a data, if you want to fetch something we will use or we want to fetch a. Let's say a GitHub review is posted. So we need to be able to, I mean get to know whenever the PR is posted so that that's what our tools, APIs and webhooks will come into the plate. Let's say if you're looking to reading unstructured input or writing language and getting to know the intelligence generative piece, we'll use LLM.

Let's say that we want to score or anything which, where we want a deterministic point of view, we'll use deterministic machine learning or probably we want to use this. We want to look at the past knowledge basis. So the code can be huge. So we want to go ahead and say that hey, what I want to do is I want to look at the code which is similar, right? Because if somebody has posted a particular pr, I should be able to look code which is associated with this PR and even not just associated, but because of addition this code what will get impacted.

So I want to look at the similar codes, similar code differences into the databases, etc etc. So I should be able to use rag. I should be able to use, I should be able to fetch such semantic code right here, right? And there can be a point where I don't want to have a human, I mean AI, I want to have a human checkpoint. If it deals with the safety, if it deals with financial, if it deals with legal, I just want to have a human checkpoint.

So these are few and few components. Now move number four. I will think about what and how much of this system will do it on its own versus how much it will defers to a human. For example, and I always tell this, tell this to people is this is not a default. In general, you cannot say fully automated system.

You need to be able to think about that. What if something goes wrong? What is the consequence of your error? And I have a golden rule which I say that if your AI has to deal with, if your machine learning systems has to. An AI systems has to deal with money, financials, legals and probably your health, you have to have.

You cannot choose to have a full autonomy. It completely depends on your organization and Whenever you're thinking about designing any system, think about what happens if the error comes in. And I will tell you very honestly, even if you get 90 thing out of hundred, you get 99 things right, and that one error is right there, which is and kind of enormous consequence having that error, you'll be doomed. Your system is not right. You need to be able to choice and design by choice is telling.

Okay, this is how much autonomy I want into this particular system. And then move number five is we have a saying that everything will break. And this is a life advice as well. You need to be prepared for everything going wrong. You have planned something.

Assume that if everything goes wrong, you should be having some backup plans, you should be having some rollbacks, you should be having some fallbacks, you should be having all the plans ready, whatever happens. So similarly, we'll design the system and we'll go through every components and we say that okay, this will break. For example, what happens when the your. Your LLM hallucinates? What happens when the API times out?

What happens when we are not able to GitHub API is down. GitHub webhooks are down. What happens if the our agents work giving some reviews actually conflict with each other? We'll talk about what does it means to parallel agents in a bit. So I would ideally want.

Let's say, let's take an example. Let's. Let's take a very simple example is if. Let's say your LLM hallucinates, right? What will you do after that?

Right? Let's say if API, if a particular embedding model is not working from OpenAI. I mean it's down for some reason usage is down. So your complete system is down, right? So you need to be able to retry again then circuit breakers.

You need to be able to say that okay, I will switch the models. I will switch the models to a different embedding models. It can be anything, right? So we want our components to fail gracefully, at least not better. But don't get, don't stop the whole system to work.

It should be able to be reliable enough and this is known as the reliability engineering is to be reliable enough so that our models actually perform really well. Now these are few things. Now there can be multiple failure modes and I will talk about few of the failure modes and then we'll discuss it in detail what will fail in our project of AIPR review agent, the first one itself is hallucination. For example, the model will state something but falls in a place that matters. Which means Hallucination and I don't disagree with you and I always say that hallucination is a feature, not a bug.

But we'll take it. Hallucination is definitely a consequence. If it happens it's going to be very bad. So the way I will design is against is that everything which it should produce, it should have a citation requirement, it should have a fact share layer. It should also give the confidence if the confidence is below a particular level, you want a human review to review that thing and I will designing it against these requirements.

Then let's say that your model drifts has happened. For example, let's say that you were building some. You build some AI agent. What happens that it was extremely good on your evaluation data set. And again we'll design those evaluation data set.

But let's say that was amazing at your whenever you were building. But as soon as it went in the world things changed. What are the prime example of this particular kind of system is in this LLM case it might be prompt. But if you're from the mlops and core machine learning background, you know about this model drift. For example, let's say that you built an email spam detection system, right?

What happens? Suddenly the hackers started changing their strategy of how they spam you. Suddenly the hacker strategy started changing how they put up. You put. You put your system into wrong place.

Now it was good when you trained and we build that system. But suddenly things were not going well. So I will have a monitoring dashboard. I will have alert thresholds. I will have a periodic training.

I will have a periodic prompt updates. I will have a rules fallbacks. I will, I will just sort it out for the common thing. So I will have those designed for that kind of problem. If my tool and API is timing out, I will have an external system.

I will have a timeout and retry. I will have a graceful degradation on the partial data. I will have a circuit breaker on a dead service and I will discuss about it. If you don't know about this timeout and retry circuit breakers and all that, please don't worry. We'll be going through once we reach that component.

For example, let's say feedback loop poisoning. For example, let's say somebody given a feedback to your AIPR review agent, what happens that that particular agent is not looking at your feedback and improving further. So we need to be able, we need to be able to look at and say that hey, you need to improve yourself. So we'll have minimum evidence threshold that okay, you eventually will use Those feedbacks which is provided now also there are times that bad feedback is stored. For example, let's say a very junior engineer comes into the place and then reviews a particular PR into that particular tone in a very bad manner.

Then your agent stored that feedback and then improved it. So we want to be able to make sure that the feedback is stored as good. Right now there are times when we need to also change. So if a particular feedback is not being used too much, we also need to decay our old feedback. Right?

We don't want to use old feedback which is no more helping and increasing our storage of our embeddings. We don't want to do that. Right? For example, they can be orchestration deadlock. For example, let's say that you're having two parallel agents coming together and they don't disagree.

So what you say you eventually don't work? Let's say that. When. Usually when we talk about the parallel agents, you'll get to know that usually whenever two parallel agents comes into place, both has to give the answer. And then we have a merger which combines the result of both of it.

Now what happens if one of the agent do not give the input. Output, Sorry, right. So your merger receives just one input or maybe both of the agent do not give any input. Right. So your merger is also failed.

So how do we do about it? How do we make sure that our orchestration deadlock is not happening? There's no dead end. We want to be able to build our own system around that human bottleneck. For example, we want to make sure that our, our we have our auto handling it most of the times.

But we want to make sure that a humans clear set. I mean escalation. So if it keeps on escalating every single thing, we don't want to do that. So we should have a priority look at the priority PRs. If, if at all the HITL eventually comes, we want to have the priority pr.

For example, okay, this is what you look at. So we need to do the capacity planning and this is an important problem. And capacity planning, which is. Okay, how about, I mean we are having let's say thousands escalations per day. Can my human review it?

No, absolutely not. So how do you think about it? How do you prioritize? If it is really a thousand escalations per day, how do you think about having a queue prioritization? How do you prioritize your peers according to the business requirements?

Right? So these are the human bottleneck. Like you cannot have a technically one human to Review thousand first off in a single day. So you need to do a capacity planning, you need to look at the queue prioritization, which ones looks at the good one and another one is the almost right problem. The most of the times I feel that your agent is almost right, which means 90% correct, but it is kind of 10% subtly wrong.

So we want to look at, okay, we want to flag the low confidence output, we want to look at random audits. We should be able to doing maybe every two days, every three days to our system. We should be able to rotate our reviewers, for example, that we should to be able be changing our models in general. For example, we should be also checking with wrong inputs. We should be able to check with new data, new inputs to be able to see that if it is really performing well or not and then test how well it is performing.

Right? So these were few failure modes. And we'll discuss it in detail once we talk about our system. Still, please remember we have not came to the system as of yet. Okay, so now talking about that not every system eventually needs the same level of human involvement.

And I always talk about if a full automation is required, you want it to fit where it is a routine task, it is a reversible task. The task is of low stakes. The human over there can review it periodically. If it is a human reviews output, which means the system produces and human verifies it before it goes out, which means that we are dealing with the reputational stakes. So it just makes a draft and then, and then does not sense it and waits for the feedback.

Human handles the exception which is going to be the. The one which is going to use in our system is system handles the easy cases and human sees the hard one. For example, the one which has anomalies the system which has a low confidence, right? Human decides and system prepares, for example, systems gets the all context and then human makes the call. For example, especially in the financial decision, the this is an amazing stuff where you take all the data of a particular.

Let's say you're providing a loan to a particular person. You want to gather all the information and then get all the scores. And then finally the human is approving whether do it or not. And then we have another one which is full human with the AI assist. Now, if you understand what we are doing and I want you to work on this, any system which you're going to work on, that is what is going to be the consequence of error, right?

So let's say a wrong style comment is annoying. For example, it can be any stuff which is right there. But if you miss the SQL injection, which is going to be dangerous for your system. Right, that's going to be dangerous. So if we are able to, if our agent is giving a wrong style of comment, it's fine, that's not much of an issue.

But if it is missing a crucial security issue while reviewing the code, it is going to be very dangerous. Reversibility can an auto posted review can be disputed and removed but a merged migration cannot be unrun easily. So what is the kind of reversibility for our kind of system? The system maturity which means new systems need more override, proven ones earn less. So is this a system already there?

So we need to give him more human, right? Right. So these are a few factors, for example, the consequence of error, reversibility and system maturity which you look at designing any sort of system. Now if you start with more human involvement than you think you need, you reduce it as the system proves itself. And that's what we mean by system maturity is let's say that you build the system and you increase the.

And I always say to the people it's fine even if it takes a lot of human to build an agent, it is good actually because you're going to looked at your agent very, very consciously and as the system matures over the period of time, you reduce the human involvement rather than recovering from removing it, which is rather than making expensive mistakes. And I'm not telling this just because if I'll tell you a real story, at second relapse at SBL we had an amazing conversational agent which used to go on LinkedIn and have a conversation with the prospect and we had a very good big client and one of the one of the biggest in the United States of America. And he eventually came in using our platform and what our agent did, our agent reached out to his prospect and customer and then he scheduled a dinner and the person was in Netherlands. Basically the customer was in Netherlands. But I mean we did not imagine that our agent will set up a dinner.

That Netherlands guy eventually was a good founder. He came for a dinner and then maybe after nine hours that particular guy got to know understand this happened in less than nine hours, all of that, which means that even if you would have happened at least two days later or three days, that that was our assumption, right? That even if it sets any dinner, it will take at least two to three days to schedule anything and by the time we'll catch it up and then we'll say no, for that meeting or reschedule that meeting, but nine hours, we would have never believed that would happen. But that happened, we got doomed. And after that of course SPL improved itself on more grounding responses.

And you will talk about assumptions registration towards the end, why assumptions? What actually happened? That okay, our assumptions failed, which means a system failed because our assumptions failed. If we would have actually made the system in a way to say that no, anytime, even one hours, even things can be set up, we would not have made the mistake which means a system failed because an assumptions failed. So basically having the right assumptions about your system is going to be an extremely inexorable, extremely important problems and design issue which would be having towards the end.

So the way you think about it is cognitive design. And that's what we're going to do in part one is spend a lot of time in designing the cognitive structure. Then how do you do the system architecture? What is your architecture? How do you retry, how do you how to make it system reliable?

What technology stack are you using? And most of the time I see people, they're like hey Ayush, let's just go ahead and use, I mean we'll build our system using Lang graph. Let's just go ahead and let's use postgres, you know, a queued rant for retrieval. Let's just use any vector databases. But there is a certain cost and gain in deciding what technological stack which you want to use.

For example, when we talk about choosing our where we'll store our memory, we talk about three different databases. So the cost is obviously cheap. But maintaining those three different databases, for example reliability, failure modes and all that thing is going to be very expensive. So we want to choose something which is a little bit in the cost but actually does pretty just. We need to just maintain one data store rather than maintaining three different ones.

And then why are we doing that? What is the gain of that? What is the sacrifice of that? That's how we choose a particular framework. Framework.

Choosing a particular framework is not a big deal. Frankly speaking, people choose. I think people should not be dependent on the framework. Frameworks are very, very easy to learn. For example, we'll see two types of things.

Which data stores to use and how to store and thinking procedure to eventually what to use. Then we'll talk about okay, which orchestration to use, whether to use langraph, whether to use a temporal. And then we'll talk about okay, what architecture level we should use. We should talk about modular monolith architecture or should we Talk about microservices. What happens if we do it Microservices right now?

What happens if we do it Modular architecture right now? What happens that okay, if at all, then we talk about, okay, how easy it is to shift to another framework. We don't want to design our whole system around one framework. So all of that is our system architecture. We want to build in that particular manner.

And then we have this other phases which we'll talk about as we go. All right, so now what we'll do is now you understand a little bit about the mindset which we are going ahead with. Now what I will do is I will give you the first principles itself. And I always say that first to design the system, to understand what we are doing. We said okay, what if an AIPR review agent is not there?

What my human is doing, My human was technically we were able to create a steps where it was failing, how it could have failed. We write down every single thing. Now we need to think about is before even we do that, we need to think about now we'll start to implement every design template. A mindset which you produced is from the first principles is why the system exists, how a senior engineer will review a particular PR and how do we map the mess to an agentix system, right? And then what kind of memory.

So you will see that transition happening from our mindset to actually writing our design. And please understand, most of the times it's just basic look through. That's why the experience matters, right? You want to understand and you want to interview that person who's doing all in all. So let's go ahead and work on our part.

Number one, the first principles is going to be a very interesting conversation and I'm going to take a probably a lot of time discussing about the first principles. So whenever, whenever we think about, okay, hey, let's design a system, let's say a PR review agent or let's design an A system which or any kind of agents most people will start. Okay, how do we build it? Okay, this is. We should put a rate limiting, we should put our security, we should put observability.

All of that is very, very important. I'm not neglecting that that level of system design. But the way I think about this, a system design has a lot to deal with. Product engineering is that you're literally until unless you have a specified problem. But again, if you're designing an overall system, it again comes down to a very simple thing which is how do you engineer it?

And the best way to engineer it. What if. If it is not there, right? Whenever you would decide designing an sbl, right? Even sbl, okay, what if SBL is not there, which help us to design our vision.

But technically speaking, when we were designing an SBL conversation layer, we're talking about what if that particular thing is not there, what happens. So similarly what I want to do is I'm going to talk about is why it exists at all. And if you answer it and once you answer it, the architecture becomes almost obvious and, and my students will put that in comment box and they will know it because if you remove something from here, you won't get it right. So you will obviously feel the pain. And as I say, necessity is a feature of invention.

That's where is the mother of an invention. So we'll create that necessity intentionally. Now the thing about it is basically why the system exists at awe, right? Which is let's say that you have a team. And obviously this is a very, very basic understanding that what happens if a team is without an automated review?

So if it is a big team, every pull request waits for a serial engineer attention. For example, you cannot get it if you're fail any mid level or junior engineers. You need to be able to make it review view by the senior. But the resource you have will be very scarce resource. For example, PR will be queued for hours and days.

And I have honestly noticed when I was at my days at Artifact or zenml and other search organizations, it used to take me at least a week to get proper comments on my pr. Now what companies are doing, they're having code rabbit type of PR review agents giving them the instructions of how they want apply particular PR to be a reviewed and then that particular agent is at least doing the basic job in figuring out okay, this is where you need to fix. So basically I mean that's what. But previously it was pretty slow, inconsistent. For example, if there is a several engineers they might be looking at the same issue or another issue on another day, etc.

Etc. And then let's say, and I'll tell you, let's say that you start one setting and then you start review and doing some work. By the time you go to the 10th, you will notice obviously that your fatigue, your basically ability to do with the same energy, with the same intelligence, with the same mind at the tenth point will be very low. So what happens because of that inconsistency occurs. So that's what the fatigue means is the 10th review of the day is not of the day is not the first.

Which means the 10th review is the intelligence and the level and the quality of review is not equal to what it did with the first. Because human has a tendency way to get fatigue, right? And if the cost and the cost if no reviews happens is that, you know, people spend a lot and lot and lot of time in doing that thing. So if we remove. Let's assume that if we just remove this and we can feel the problem.

Similarly, whenever you work on any sort of system, remove that thing and then think about what the problem might come. Then think about, okay? An automated reviewer exists to solve exactly one problem, which is reclaiming senior reviewer engineer attention by automating the mechanical part of the review. So that my human part is reviewing at the actual thing, which it needs to, rather than doing a lot of basic stuff or probably something which could be. Obviously could have been caught or some automating the inconsistency or to not have the fatigue.

So technically, we don't want to build a system which does an automated peer review. Instead we want to have an agent. What it will do, it will reclaim what senior engineer attention by automating the mechanical part of the review. An extremely, extremely important statement. What we technically did when we talked about this particular issue of what is happening technically, the issue is not, hey, I just need automated pr.

If you notice, the issue is all about the senior engineer. And this is one such example. There can be multiple, I mean examples. But in this example, it was a senior engineer's point of view that hey, I want to get or reclaim my stuff. Even, even if there are multiple engineers, they can be inconsistency, there can be fatigue, it can be slow, it can be.

If the cost of no review happens is extremely huge. So technically the problem is not to design an agent which does reviews in pr. The problem is that how can I design an agent that reclaims my senior reviewer attention by automating the mechanical part of the review? And human expense is only the judgment where it is genuinely required. That way your inconsistency will not happen and the fatigue will not happen.

Because if the person had just had to review the couples, right? And then keeps on giving feedback to the agent so it improves further. Now, a review agent is not a replacement for the human judgment. And I always say this to everybody, whenever you design the system, don't think about that. We have to automate every single part of it.

We want to make sure that human judgment is very scarce. Can we, can we, can we do one thing is put up this particular situation so that human only focuses on where it actually requires. That's what the whole agenda. And this is the power of when we just listed out the problems. And if we want to look at the world word selective and again a very, very, very important word is it shouldn't our.

Our AI agent should not surface a PR with n number of reviews. Correct. It should only put out high value findings or something which is very interesting. It should not comment every single thing out there. So we should have the selective review from our agent on the mechanical part so that.

And if it is possible just give the high value findings where you're very confident and if you're uncertain, we will escalate that to a senior engineer. And selectivity is the seed of this whole architecture, which means what will be given out of this system? What will be given out of this agent? Why it is saying that? How are we making sure that they're saying the right thing?

Where are we putting hitl? Where are we putting human in the loop? How are we observing it? How do I making it secure so that it does not gives out anything bad. How do we make sure that lot of failure modes which we discussed how to which we also discuss again.

But how do we make it so it's all about how do we make it so and so and so selective so that it works pretty well. And again, if we need to carry one thing after this section, which is L0 is the system optimizes for surfacing findings worth a senior attention and differing the rest not for maximal output. It should not just go ahead and blunt out the feedbacks. It should be only selective, not coverage. Is the first principle altogether.

And the way we came to this particular statement is say that okay, all right, let's remove what is not needed. And then you will see a new problem coming out of it. A new ways, a new way to think about a new problem framing will come out of it. So usually most of the system design has to be reframed in a particular situation. And that is going to be one of the most important.

That's why the first principles exist. So now we know, okay, we know, we understand our problem statement. We understand this is what we think about it. Now what we want to do is we want to think about how a particular human would think about is how a senior engineer would review, right? So that's what I told is write down every single thing which a particular person would do, right?

So before we do literally any technical part is when you're designing something which is looking genuinely hard most of the times is just how brain as we have already solved it in the real world. If you technically see what we are doing, we are doing okay. You can use this agent to find the pr. People were reviewing PR long back as well. Prepare an agent that creates content.

People were creating content prior as well. Create an agent that approves a loan. Okay, people were approving the loan before we want to do one thing, we want to look at the already solved problem and redesign it to make it, I mean efficient, automated, or maybe where a human could actually just focus on what is judgmental rather than the mechanical part of it. Right now they do four things in a native which, which does not. I mean, if you just look at the prompt reviewer which will based on the prompt it's giving out.

The first one is bring the code base context, which is an amazing statement is whenever the particular senior engineer is looking, they're looking at the knowledge base to look at what contradicts the past decision. If it's something based on architectural stylus, something based on decisions which this particular thing has not been done, then the reason across separate concerns. There can be a security. Is it secure enough? There can be correctness.

Is it correct enough? Is the test has it covered the test or not? Whether documentation pass, whether it is readable or not. Each with a different mindset, which means that they're thinking from a different different mindsets for reviewing a particular pr. And then they stay skeptical, which means they do not assume the difference is correct.

They cite the evidence which is this is wrong because line 40 can be null here, not looks off. It's not just about, you know, saying that, okay, this assume the difference is correct. No, they can say that this line can be impacted because that line can also be impacted. So we want to make sure that. And citing evidence is making sure that, okay, this can be impacted.

Not something this looks off. Nobody can review and fix it. If it just looks off, it should be a very, very good evidence across it. So if you think about it, what we are technically doing, we are seeing bringing the code base means the agent needs retrieval so that it retrieves the right code base context from the whole repository according to what PR was made, right? So if the PR made so we want to look at the things which is similar, but also look at where it can also impact that code base.

Then we something we have known as context rag and all which we'll discuss, then it should reason across separate concerns, which means that we should not have just one agent to talk about. We should have a Several agents, such as a security agent, a test agent, a documentation agent, a correctness agent, four or five different agents, right? And then there should be some master agent which will take all the concerns and put that into up and combine their results. So four people reviewing and then there's one orchestrator which reviews everything and then there's final one which combines the all the agents output. Right.

Then we want to say okay, it stays skeptical and cites evidence. Which means every finding needs a rational and a confidence to say that. This is why I said it. Now if you really notice, we have built. We have certainly did not say okay, let's look at the context and all.

There was a thinking framework to come to this particular line. The thinking framework was okay, how a senior engineer will review. This is how we'll review. So can we write that into a statement? Right?

For example, there can be a security concern. Could this be exploited? They can be quality concern. Is this logic correct? Or according to my database, what's untested missing edge cases, untested brittle assertions, coverage gaps.

The docs agent will the next. So there are four different mindsets which is looking at the same time, right? So now first we'd say that the problem from our architecture is we're looking for something that actually selectivity which means looks at the high value findings and optimizes for surface findings worth a senior attention and differing the rest which means less. I mean high confident PR should be reviewed and uncertain should be given to the senior engineer. Alright, so in this L1 we say okay, now what I'm going to do, we're going to say four specialist concerns born from how a human reviews.

So we're having security quality agent, security agent, testing agent and docs agent. Security agent will say that hey, I'm going to test it whether it's secure enough or not. Call the agent where it matches. My logic is correct and right according to my architecture, design patterns, writing code standards and all. Is everything covered?

Is there any missing edge cases etc. And can other people people read it? And this is going to be how we'll design our multi agent system. But now we are still not there. Now our next step was what according to our design template is mapping the mess is what happens on a PR today.

So a developer pushes a comet, then it opens a PR and then waits. Eventually a reviewer will notice from some notification context switches into the change so that it looks at the whatever the changes happen, reads the differences, sometimes pulls the brand to run it leaves the comment and then the developer Reiterates. So what is happening is if you look at the very specific weighting is a pure and pure. I mean cost and context switching, which means switch the context into the change. We'll talk about context switching.

What exactly does it mean? With the help of an example. But if you want to look at everything a particular guy will do, you want to name two things is what is the trigger and what is the output. And trigger is very simple. A pull request made.

So GitHub actually offers a webhook whenever a PR is opened or not. GitHub emits a pull request. So very precise. Because we just want to know whenever the PR is open so that we can just go inside it and pull out the code differences. Right?

Pull out the whole repository and then pull out the comments from that PR guy and then use it for the review. Right. So my Input will be GitHub emits a full request whenever PR is open. The output is a single structured review posted Back to the PR using GitHub pull request webhook with findings attached to the specific files and the lines. Now please understand a very very important word is this.

The word known as structured is a very, very, very strategic work, which means we are not just saying that hey, it's a blob of prose or anything of that kind. It is a list of findings, each with a shape, which concern raised it, how severe it is, where in the code, why and how sure the agent is. We are not just saying hey this is wrong, we are saying where did it come from? Traceability, how severe it is, where in the code, evidence, citing evidence and why. That is the reasoning behind it so that we can ensure that this is the right reasoning and the confidence behind it, which is the evaluation stuff, is how confident it is so that we can choose to escalate this to human or post it.

So if you look at the first thing which is agentic type, which means output will contain agentic type. It can be security quality testing which concerns raised it, which means what are the concerns? It was a security concerns, was it quality concerns, Was it doc's concerns? Then severity plus category what is the criticals? If you have seen the demo critical or information for example, you know, if it is a security issue then it's a critical.

If it's just a normal docs issue, then information that this is something which is already there, right? Critical might be a missing edge case, very important edge cases. So we're looking at what is the severity. So in a particular review, which will come out of it, which Concerns raised it, how much severe it is, how bad and what kind of is and file line and the exact location, which means where it came from, what is the line, what is the exact location so that we can look at exactly where it is and confidence preparation, which is how sure it is and why. So that we can first of all confidence which will drives the human review gate.

But what exactly what happens if the confidence is wrong itself? It says 90% confident that let's post it. But suddenly we found out that oh, it's not good. So it should. It should have given to human.

So we should have the rational. So that it also explains why the reasoning behind it. So that when we review why it went wrong, we should be able to go find what exactly happened, which is basically makes the finding auditable and disputable and trace it back what was the reasoning which was wrong? Right. So if I generalize what we did over here is what is the precise trigger, what is the precise output and the shape of the object that travels between every component.

This is the shape of our output, which means what is the agent type, how bad it is, what is the exact location and how sure and confident it is and why? Right. So if we carry our architecture after mapping the mess is the Trigger is the GitHub webhook web. Webhook will say hey, here is the PR. The output is a structured review and the unit that flows through the system is a finding is output with the agent type, the severity, the category where the files confidence and rational.

All right, so how do we think from the industry standard? So if you think about what current people will do is they will just use the difference of the code. They will have the LLM verify the difference and then they will put the comments out. But we just identified that we want to be able to do several types of reviewing automation that does not work. It's good for a demo.

It looks for a good demo that LLM is producing something, but there's no mechanism to it. First of all, we'll do several type of pour runks. The first one is linters, which means it cannot reason. So do we use linters basically looking at a pattern match syntax or style rules to make sure that we have a proper linting of a particular review, which means intent, the logic and all that. Which means that it cannot reason about anything.

It can just look at whether it is consistent patterns and styles or not. Not even reasoning about intent, logic and whether it tests. It's just there's no semantically. Then you have another type of review automation is static analysis which is data will flow and the type analysis and then it finds the real bugs. Now it might put out anything, which means that it might give you that you put several test cases and they'll find some real bugs, which is an extremely good one.

But where it will fail, it will fail. High false positive rate, no code based wide judgment and cannot read the documentation. We just know that there is some bug which is still people might be doing. Then we have another type of automation where we single LLM review which means one prompt judges the whole difference. What will happen in this?

Pause the video and tell me one mindset of four concerns. No grounding in the repo and hallucinates with confidence. We don't have any auditing. We don't have. You know, there is no.

There is just one agent which is thinking about from four different angles. It's just one. One guy. So as you know, if you give an agent too much of work, it won't perform well. It does not know about anything about repository.

It will hallucinate because the context will get bigger. Then there's no auditability, there's no observability. So the another agent is agentic. Fan out. Can we fan out and fan in?

This is known as. This is one of the agentic design pattern which is known as fan out and fan in pattern. What does it mean that you roll out a specialist agents for example to review a pr? We are rolling out four different specialist agents security docs, tests and quality. And then we have fanning in.

The fanning in means that okay, we are combining by an aggregator. Aggregator. So this is the system which. Which I want to use. Of course I can integrate this.

Of course I can do this. Of course I can do this for any small task. But this actually stands out. But what is an issue we wanted to be able so that it can orchestrate. Well, step by step, make sure that it's reliable, make sure that it's secure.

Make sure that is. I mean failure proof. Make sure the retrieval is nice. So we need to design a retrieval system accordingly. Make sure that we store the memory somewhere, etc.

Etc. Right. So the way that we carry out is we'll be carrying out a parallel specialist. Not a single prompt. Right.

Okay, then the grounding problem. And again we are talking about. We started with saying our senior review agent and then we figured out. Okay, now we are what. What we're doing.

We are mapping that every component so that it can actually writing the solutions for it. So what is a Grounding problem. In the mess we saw, a senior reviewer does not have this problem because they. They know the repo, right? Because they already know the repo.

They have the context and everything. So if we give our agent a particular situation, which means, you know, a difference of the code, right? Can we give the full repository in the prompt, which means can we give that agent of my full repository? Absolutely. Yes, definitely we can give, but we cannot put our full repository in the prompt.

So what we are technically doing, we are saying can we give this ability for our agent to know my repl before even it reviews? So if you put out everything in the prompt, that will exhaust the context window and it will pull out an extremely bad thing. So what should we do about it? The answer is retrieval, which means for every difference, the difference is basically. I will again explain what is the difference?

Difference is basically what you have added plus code or a minus code compared to your actual repo pr. I mean repository code. So you see that plus green signs and minus red signs. So that is a difference for that particular review. Can we fetch only the most relevant slices of the code base and put those into the prompt?

This is also known as the context engineering. And we'll talk about this in detail. How are we putting it? Okay, so can we do this? So the grounding problem is solved.

So what we are doing, we are looking at every specific problem from that mess and then we are solving it. So retrieval is what will turn the stranger into the colleague. Which means, hey, it is going to give you the ability now what? And you obviously know this right now, what kind of review or what kind of memory does review need? For example, we said that we want to look at retrieving the context.

The grounding problem said we want to retrieve the context, but it can be a multiple. For example, if you think about from the engineer perspective, the engineer can have multiple memories. It can be semantic, which means the code base itself, which means the functions, the classes, etc. Etc. Which is right now, conventions, architectural records and all.

It can be episodic, which means past reviews, what was flagged before, what was disputed, which means that before this pr, something would have happened. And there can be a procedural review. Procedural memory, right, which means how this team like things done, which means what is the convention? It's like a procedural memory in general. So for the semantic, we want, for the code base, we want a vector embeddings in the similarity search.

So that what we can do that? We can, whenever a code base comes in, we can just compare the vector embeddings with our code base embeddings and pull out the best ones episodic. We want a timestamped relation rows. We'll talk about the memory again in detail when we design when we do the data data engineering flow. Because data engineering is going to be the very important part that is the next part after the first principles.

But timestamp relational knows that in 2021 we could have done this but recently 2022 we would have done this right. So you want to look at. At this after this point of time what has happened, what what was flagged so that it can know about my episodic memories and then procedural. For example I like to eat dosa right? Is a.

I mean somatic okay, this, this, this. This person is likes to eat. It's. It's. It's eats DOSA because this has a protein and all.

And frankly speaking I'm going to continue to have this right. Similarly it's a very small and high priority that how I like things to be done. I like to think about reviewing a PR in this particular manner. That's my procedural memory. So every time a new different PR has to be reviewed.

We want to make sure that we are context engineering it right Using these kind of memories right here. So what we are saying, we are saying we are Semantic memory wants similarity search over embeddings which means a vector store so that we can pull out the best ones best code which is similar to what we want. Episodic memory wants a time ordered curable rows and procedural memory is small and structured which means facts and the rules and we have to write it. So we need to figure out where are we going to store the semantic where are we going to store this episodic and where are we going to store the procedural data engineering is the most and important part of the whole system. So if you think about there are three types of data shapes.

The first type of data shape is itself is our semantic episodic and procedure. So we want a semantic memory of the repo which will have our vector vector store. We'll talk about what text stack and what we'll be using past reviews and findings. Want a relational shape which is time ordered stuff which means basically timestamp relational rows then conventions and decisions. Want a small structured shape which can be again so we'll discuss about this again when we design the databases.

Alright, so how do we trust basically when we talk about a human it's very obvious that okay, you know they have a you they say that okay, at this line. At that line. So suppose the agent posts a finding this endpoint is vulnerable to SQL injection conference is 60% a developer disputes it. Now what if there is no record of why the founding was raised, which context was retrieved, which prompt version ran, what the model returned, what it costed. We cannot defend itself, we can.

It cannot be debugged, it cannot be improved. If something can be measured, something can be improved. But if we don't have which context was used, what was the version of the prompt, what was the model which was what was the embedding model, what was the retriever which was used? What what happened at that point of time? We want to trace it all along.

So we want to make sure that we want to improve the system. So we want to audit which means without which is trust will require third thing beyond reasoning and grounding known as proof which means for every action the agent takes every span of work, every LLM call, every tool call, every decision, it must be recorded as an event in a time order durably which is one of the part of our observability plans. And see, I am I have not told you why observe the word observability directly. You have felt it, why it was required and that's what the beauty of feeling the problem and then doing it. Now, the single stream of events is what powers three things which is a trace viewer.

We should reconstruct any review end to end. If it has reviewed anything, we should be able to know what it did, what it achieved, the way it did every single traceability. We should be able to have an audit trap. It should be able to defend or dispute any findings. And we should also have economics, which means how much it is costing to run the entire system the cost economics.

While I'll have a separate designs coming all together in future talking about economics token optimization how to design by keeping the token optimization in the mind. How to design by keeping the cost in the mind. How to so here we are designing in a very structured format. And see it depends on the business. Today business says cost is not a problem.

So your idea should not be on the cost. But let's say tomorrow somebody comes and tells a cost is going to be big problem. So even though you design the system, you'll design you'll design another deck. Talking about this is how I think about after designing system. Okay, now this.

Now let's design a system for optimizing our tokens which is caching and all that which is one of our thing which we'll Talk about in the later phases. I mean, not in this particular design. Maybe in the future designs. We have a lot of designs coming. Which is, which is going to be amazing.

Okay, so we want to have an event spine so that we can trace everything. So what technology we can use any observability tool. We'll talk about that in a bit. Which means that we want to make sure how much is costing, what is the audit trail, how do we trace it? That is a fourth data, which is, which is time series in a particular shape.

We want to look at every single thing carrying cost, latency, confidence and outcome so that we can trace it. Now, if you need to trust it, we will be also figuring out when not a trust is. Because if you remember, our problem problem was we want to make sure that our agent does things right. So when not to trust it, your level 0 said, L0 said that system should be selective. L6 gave it the confidence field, which we trust and prove, which is how much confident it is.

Now if we say okay, anything beyond this confidence, we'll put this to a particular human, we'll put this to a particular. Escalate this to a senior reviewer. So if the confidence, if it is no critical, the action is post automatically. Post the review automatically. Maturity owns the autonomy.

Even, even initially, if the confidence should be at 0.6. So anything below 0.6, you want to go ahead and you know, put that as an escalation, that's fine. But over the time it should mature confidence below threshold, which is route to a human threshold approval queue. Any critical finding escalate, which means literally escalate any critical findings. Developer disputes are posted, which means the posted finding is not correct, which means record the feedback, which is reversibility and learning loop.

It should also learn and then improve over the period of time if somebody disputes that the agent answer is wrong. So we will implement the HRTL gate, which is human in the loop. Then we have also discussed there are multiple failure modes. What if the hallucination happens? So for hallucination we have the grounding so that we have the right retrieval, we have rational and then we have the confidence.

So we are trying our best to not get this failure. Then we have this tool and API timeout which is retrying with backoff. Then we have circuit breakers, then orchestration deadlock, which means the aggregator waits forever on a hung agent, which means our parallel agents 4 agent, they are not giving any output. So my aggregator is waiting. So there should be a timeouts on every node.

Okay, so that it should timeout after a certain point of time. You have always seen how the timeout works. The almost right problem, which means the finding 90% right but subtly misattributed. So we'll be having again confidence threshold hitl right here as a defense human bottleneck. The approval cues grows faster than the reviews clear set, which means this.

We should have the escalation rate monitoring on the event spine if it is too much of escalation which means that my age in needs to increase its intelligence. So we'll be having that monitoring so that my escalation is not too much. And we should also the capacity planning if at all agent learns a wrong preferences from a few disputes. So minimum evidence threshold before acting on a feedback, which means some feedback was given from a wrong engineer. But later not every feedback is good.

People say right, do not take every feedback. You should take what you what makes sense. So we'll be having evidence threshold before acting on the feedback and then which is the invariant cap which means let's say that GitHub tried once, but it failed. But then it retried another time, which means that it can post two reviews at the same time. We don't want two reviews at the same time if the reviews are similar, if two reviews are same, if it is not already into our system, we will not go ahead with it, which is we want to dedupe dedupe before posting, which is making sure that we are not retrying on the same webhook which is the reliability layer, which is retries circuit breakers, item 10 CDUP at the algorithm.

Everything mapped to the failure most. Please understand. You have not told you why reliability is needed. We just talked about the problems. Solutions came automatically.

Okay, so finally we are getting towards the end of the first principles and I hope that you really are getting it. Before we even think about designing our data layer, what we said is a pull request triggers the work. It is enqueued because the trigger must be acknowledged. I'll talk about what is it? It is enqueued because we're just saying okay, it should be looked from the GitHub PR GitHub webhook sends it, we store it somewhere.

We'll talk about this very, very important statement in a bit. After the data part, we'll talk about this statement. An orchestrator fans out the work to four specialists which is running in parallel. Known as one of the design pattern we are using is fan out and fan in. Each specialist is grounded by retrieval over the code base.

So our we Are grounding it using retrieval design. Because it will not hallucinate. Because if you don't ground it will hallucinate. That's why the rag was invented. The code base, the past reviews and the conventions are three kind of memories.

Which means that we have three data shapes by which we should be grounding it. Each specialist returns a structured findings with confidence and rational. Then aggregate mergers merges then and deduplicates them and computes an overall confidence. Then it applies whether it needs a human or not. Then posts automatically when confident routes to human, when not.

Then every action along the way is written to an event spine so that we can trace everything and the whole thing can be traced and audited and priced. And then we have a reliability layer which keeps step degrading to slower but correct path. Right. So we talked about three important databases which is memory. Sorry, we have three different memory, semantic, episodic and procedural.

So here the code base, the current code base, the past reviews where we'll store the past reviews and the conventions which are procedural memory. So where shall we store our these kind of data sets in which data data shapes and how do we design our data layer? And this is going to be another important part. And we'll interrogate which is you might be obviously telling okay, for who for storing. You know, our code base will use probably some vector store.

Then past reviews will use some postgres using timestamp. Then for conventions we use another one. You might be noticing that it's a huge problem there. Which means okay, four, but why four? It looks good.

Okay, we'll be having four different stores. They all had very nice capability. But what happens is a lot of times, as I already told you is each comes with reliability, security management, maintenance and all that. If one fails, everything goes gets gets around the way. So how can we minimize further from the failures perspective?

That's what we're going to work on. So we'll talk about that with. You can just see quickly what exactly we have built. We have talked about till now. Then now if we have the principles how can we design our database and using and how do we select a particular technological stack?

And then after data we'll talk about how do you, I mean assemble all the architecture which we thought about it. All right. So now we eventually went through our first principles understanding and how to think about the problem. We need to eventually come about data engineering. And we call this data models.

And you can call this any type of shapes because we talked about one of the most important layers in our Problem and in our system is how do we store data? Where do we store data? We have several types of data, so where do we store that? Right? So we'll address that data model and then eventually we'll try to assemble it using an architecture.

So it'll be pretty short, but I'll take it ahead. So with respect to what we want to do. So basically we saw that agent has over three steps stages in general, the first shape is our memory. Now if you look at that memory, it has a chunk of code and past reviews and conventions that will help the agent understand a new difference. So basically anytime a new PR is made it.

It needs to look at the chunk of code or something which is. Which it can call. So let's see if. If a new PR is made right it has some code it want to pull out all the related code for that particular or anything which is related to that particular code, right from your repository it needs to take out all the past reviews or conventions which might be related to, you know, those kind of. We call.

We can really call that as a. I mean searching for the right memory from your vector database. Now the second one is the truth. Now the truth means is basically over here is the findings. Basically let's say that you made a PR and then your PR agent is reviewed that particular thing it should be able to store somewhere, right?

It should be able to store store. What was the GitHub review ID? What was the. If it was escalated to a human, you know, it was escalated to a human and eventually you know the findings and the GitHub review and any human decisions. And third is the time for every span because we want observability for every time.

We should have every span. LLM call what is a tool called the cost, the latency and the decision which eventually happened into that eventual to debug if something has happened. And from you from our design model you understood why observability was super duper important. So these kind of things which we ultimately want to store. So there are three things which we ultimately thought to that there are three shapes basically the memory the truth truth will be the one which eventually once our agent does something it should be able to store somewhere so that we can continuously audit and eventually see the reviews and pull it up in our front end or show it in some way or maybe use that for a training purpose purpose.

So the memory is we are we are we. You know, you can use somewhere like something like QDRANT and. And. Or any vector database what it does it Embeds the code for semantic retrieval. So it uses some LLMs for the embedding and then it embeds the code chunks so that we can retrieve it semantically.

And then there is a different shape known as truth. Now what does that mean? Is the postgres. So basically for truth data data shape we might be end up using the postgres where you will store the reviews findings human in the loop rows or GitHub IDs. Then there's a third one which is the time.

Now here you'll store your observability things for example spans, LLM calls, tool calls, cost, latency, etc, etc. Now if you understand that you will be okay, go ahead, let's create a three different data sheet. But in most of the time we try to over engineer something which we should not. For example for this pr, what code did we, you know, retrieve, what review did we produce and which model calls it expensive. So if you want to answer this one particular cushion for this particular pr, we need to go ahead with three different stores.

The three different app has to query three different systems and then stitch the answer together in python. Now what does that mean? That I mean it depends on your architectural style. But the cost of doing this is the more you introduce the I mean app, I mean overall issues basically or more data shapes and more databases to be used into this. The more connections pools you need, the more backups you need to have.

For example, what if queued run fails? What is postgres fails? What is what is your one of the data data shifts fails? What if something does not happens? Why when does the grow gets corrupted?

There can be multiple failure modes for the same thing. So for simple cushion we are having three different maintenance overload to be still doing all of this issue. Now if you understand the question that is obviously is can we keep the three shapes. We don't want to compromise on the shapes which we want. But instead of using three different I mean and still not be able to split them between three different durable database, can we put it into some current store which can handle all three data shapes altogether on its own rather than having three different data store per feature.

Now obvious answer is you'll think okay, which database is more popular? But we don't want the database which is popular, we want the one which contains all three. I don't want postgres, I want all three reflexive stored all three data shapes into one database itself. So we ultimately decided to go ahead with Tiger Cloud. Now Tiger Cloud is what it will eventually help us to give a managed postgres compatible database and then we will add the extensions which needs for AI memory and time series agent because it already has the, you know, I mean Postgres compatible database.

It gives us a managed PostgreSQL compatible database and then we can further add our couple of more important shapes such as the memory and the time. And frankly speaking, Tiger Cloud eventually does that pretty beautifully. The first one is which we needed is vector search for the memory. So as I say that we, we are looking for something which can store our semantic, our code embedding. It should embed our code, our main branch and then put it up there so that it can.

So anytime a new PR comes in, it should be able to pull the relevant code and then give it to our LLM agent. So PGvector can actually help you help your postgres to store that list of numbers in a real column. The column is known as code chunks. Let's assume that it's a code chunks which is embedding, which is the embedding of this particular content. So our entire code file will be embedded.

Now whenever a new pr, let's say a charge customer adds, I mean comes in, it will look at all the things which is closest to this particular page PR and then it will take out the one. And I hope that you understand how does RAG works and I'm, I'm not a big fan of teaching you racks right now because if you're watching this video, I assume that you have understood a little bit about the rag. But what I'm saying is it will pull out all the relevant one which is very closer to this PR so that it has the right context rather than taking all the context from the, from the back. I mean everything code at once, which technically you cannot. You're only taking the one which relates or which affects or which gives the logic.

Now a lot of time, you know what happens that the, your code file can come in millions and billions of embeddings if you're really working on the real database. So the point is, okay, how do we make it efficient? So PGvector itself, I mean we have a PGvector scale from Tiger data itself, which eventually is like a fast librarian. What it can do, it can quickly give us the right set of, you know, the two that the embeddings right away it knows where it is. So PGvector scale is an advanced version of PGvector itself where it can actually add an index so that postgres can find nearby vectors.

Super duper quickly. And that's. That's one of the favorite feature which I particularly like of Tiger data and I've been working for with it for a very, very long time. And then it has. And if you want to know more about the architecture and how it is built, feel free to go to the documentation they have listed Very well then we are adding the disk Ann.

I mean it sounds complex, but let's say that when there are millions and millions of code chunks, you cannot search for every shortcut in the RAM forever. What it does diskNN gives the search structure on the disk and SSD itself and still helps you quickly towards the closest matches. So what it is saying what ptvector scale and discannon is doing is because your code can get in millions and millions and millions of lines and millions of embeddings. These two particular feature can actually and these two particular advancement can actually help you out into getting the right embedding. Super duper quickly.

So we had another shape known as time series which is basically where we want to store for example for this particular chunk our security agent reviewed it started the security agent called the LLM. The cost was this and security agent ended for this particular chunk. Then for this particular chunk, the quality end started, it called and it ended. So we want that kind of time series data so we can audit it or we can have the observability. We can figure out when the LLM call was happened, how much did it cost, how long did it took, what decision was made, etc.

Etc. Etc. So Tiger data itself gives something as hyper table which can store them as a time chunks behind the scenes. Now for example, it does, it looks like, you know, a nice little, you know, table for example, something like this, which is agent events hyper table. But in Tiger that actually helps you.

I mean Tiger data can actually store as Monday rows, Tuesday rows and today's rows in a separate chunk internally this is how it is going to be stored and that becomes another powerful data chip itself. Notice that we are not creating a new database, we are having just one, managing just one. And then you just have to design your failure modes around this itself. Now let's say when the dashboard asks for the last hour, it does not need to look for the entire database. It can look at a decent chunk instead of driving the whole history.

Amazing point. Then we have a dashboard should not be able to call raw events from scratch every time it calls. If there are 10 million LLM calls, we should be asking what did we spend today? Should not scan. We should not scan all our, you know, 10 million rows on every refresh.

So we need to have those dashboards, right? We need to be able to quickly understand instead of going to 10 million rows, we should be able to quickly estimate what did we spend today rather than going to the 10 million rows. So eventually Tiger gives something as continuous aggregate is that continuous aggregate is eventually a summary table that Tiger keeps updated for us. The raw rows eventually goes in the agent event. So this eventually becomes super duper big in millions and billions.

And you know, overall data we should not be able to score and then figure out how how many of we did we eventually have this continuous aggregate all the period of time, such as cost per minute, the P95 latency, the tokens total or any such matrix you want to store. Now the budget card can read your summary first. So basically if you remember we said we don't want to cross a certain budget the economics of building an AI agent. So instead of looking at the whole table, they can just look at this particular continuous aggregate and say okay, if it can just read the summary and say if the today spent is already above the limit, it will block your LLM. It will not let it go ahead.

So it eventually gives us one durable database, one backup story, one place to query and one identity that connects memory through the and time. So previously we have three different database. Now we have one store where we can store using PG vector we can store all the chunks events which will eventually help us to store the times, time, time series data for the, you know, the observability and then continuous aggregation and agent health and PR cost so that we also we can monitor everything very well. And even our agent for economics task they can come and say okay, did we not cross the other thing or not this right, okay, perfect. Now we also have something known as Redis.

Now Redis is basically we'll talk about this Redis, but Redis will stay because eventually we want to skew our jobs into Redis itself. So if you want to go ahead. So before we start and I will also have to make you do this particular thing. So you can go to any link and then you can go to the link which is provided. The Tiger cloud can actually give you $1,000 in credits and you can still to use it for absolutely free for your project task which shows the enterprise level project and the product which you ultimately going to build.

And frankly speaking, Tiger cloud is only used in a lot of the enterprises. And if you put this in a project, it's going to Be looking amazing. Now you have to do these steps. But ultimately I will also when I'll show you that. Okay.

If you do not know how to do this how you can instruct your coding agent to be able to tell you what it requires from your side. Okay. So feel free. So you'll be ultimately looking something like this. What is your tiger database?

What is your OpenAI key and all that we'll talk about what is webhook secret and private keys in general. So feel free to read your schemas the way the way it looks. But now I want to talk about I mean, why Tiger Cloud eventually we are using so instead of plain postgres if you were just about to store enough reviews for review rodent findings, we should quickly use postgres. But we also need a fast vector search over code embedding and efficient time series queries over millions of agents events. So what it does Tiger Cloud keeps the postgres programming model but ads such as timescale DB, PGvector and PGvector scale that can actually help you to be efficient B time series and over and be and do the fast vector search in general.

And that's why we are using this. And one of the greatest thing which we can use this is because enterprises also uses this. And the reason why we are using Q Draw I've typed Cloud over Qtron is because the vector search is not the only question this agent asks. A PR review needs vector similarity plus repo filters, freshness, exact identifier, matching review records, cost records and audit history. Which means that we want to be able to keep this everything at once.

We need to be giving everything at once which means that the retrieval result can live beside the metadata and the review trail. Ultimately your hyper tables and all that thing. Then you have this disk disk N I just discussed when you have a millions and millions of thing this particular algorithm eventually helps you to index more of the code base I mean your index on SSD while still returning the closest neighbors quickly. That's why using PGvector scale for large vector memory which eventually the other enterprises uses it. And as we discussed, the hyper tables and continuous aggregates eventually help us to one is to internally maintain the overall chunks where we can trace it back and one continuous keep us updating and aggregating the summary of tables such as cost per agent, latency percentiles, tokens and per PR cost.

So now once it makes sense, I want to go ahead with architecture and I will combine every pieces we talked about and then we'll straight Go in how to work with coding agent to be able to code this entire architecture, half everything to architect and assemble all our thing. So basically the first step which I talked about into my mental model is that there should be some trigger and then there should be some mess to fix the workflow and then there should be some output. So to think about it, into these kind of systems, we want to have a trigger. Now where does that trigger comes from? If you understand the trigger is basically as soon as somebody creates a pr, our AIPR review agent in that particular pr, I mean repository should be able to go and write a review.

That's our trigger. So technically, how do we think about it? So if you search on GPT or anywhere, you'll eventually figure out that a GitHub webhook will arrive, will get validated and then it'll be queued. So we need to create an ingress handler so that it ingests the pr and then eventually say, sure. Now let's understand in a different way, let's say that you know, you have, and this is for the people who do not know about the GitHub webhook, let's say the PR gets opened, okay, now what you do, you first of all ingest it and you'll think about, okay, how do you ingest it?

So whenever you take something, whenever you take something, you want to make sure that you're taking the right order. And if you're from taking it from Amazon, you're making sure that it is collecting the right order, right, you take the order. That's it. Now you close the door and then you do anything else. But first of all, you take the order, make sure that it's from Amazon and then you close the job.

We can talk in jest into that similar way. Similarly, if you want to make sure that GitHub Webhook is Sunday, GitHub Webhook will send everything. It will send PR. It will send like it will send stars. It will send every single event which happen into that particular repository.

But what we are interested in is we are just interested in pr. So we want to make sure that we were only taking the pull requests. So what it does, first of all, we want to verify that that particular thing is coming from GitHub. So we will take the signature on the payload and we will reject any or anyone who is trying to forger before any work. And by the way, this is one of the security thing where we are ensuring whatever is coming to make sure that it's coming from GitHub itself and nobody is exploiting our system.

Then it checks the item potency key or unaltered key. I cannot pronounce this adaptation item potency, but there's something synonym known as unaltered. So this altered key, which is basically a retry delivery is basically acknowledge, acknowledges and drop together. It's like verifying your GitHub signature on that payload and your key which is an unaltered key. It's like your otp.

You're verifying both of it, which basically is one of the defense so that nobody is basically, I mean messing up with your system. And then it will enqueue the job to Redis. And if you don't know about Redis, while I don't want to take the class of Redis right away, but it's a but, but, but in queue is basically a process of adding a. I mean a kind of a new item to the back of the data structure that follows first in and first out. Which means it's like okay, FIFA principle.

So it's like adding whatever the things comes from if they're new keys, it does not act in front of them. It adds after that so that if the new PR switch is already there, it does not comes the next time it is already there. So it eventually adds queues a job that. Okay, this is the PR which has to be reviewed where it does in Redis ARQ now, Now if I'm. I don't want to take.

I mean session right now and what does. And what you mean by. I mean what. Why are we using Redis? Why are we doing.

But basically Redis is where we'll store our all the job and it's. It's like a scratch pad where every time PR is made it will get stored at REDIS plus ARQ which is ARQ stands for the lay something like automatic repeat request. Now what is automatic repeat request is basically an error control protocol which is used in, you know, this networking to make sure that there's a kind of reliable transitions towards unreliable channels. So what it does it. It's like accepting the receiver sending acknowledgement so the sender can retransmit or corrupted packets.

Now the way it works is very, very simple. If I want to understand, if, if I want you to understand it's basically telling okay, as soon as the GitHub PR is made it is going to take. Okay, I'll match first of all that it's truly coming from the PR by matching this signature with the idempotency keys. And then it will Put that up into it will put that it will queue that job into Redis and immediately return to GitHub says I'm alright, I have received the PR request, that's it. And the GitHub says sure, close the door.

Now the reason why we are doing this and I want to think about it a little bit is why are we doing this particular is why we cannot just take the pr, give it to the LLM and then directly, you know, give it back to your GitHub agent. Why are we using, why are we queuing our job to redis now in real world, you know what happens first of all, you know a lot of PRs will be made so there will be a lot of queues. So you'll say okay, we'll increase the number of workers, we'll work on it. But, but, but, but if you think about it, even there is a bigger constraint. The constraint is very simple is GitHub.

Webhook expects a fast acknowledgement. You should truly say yes, I've received Yes, I have received in about 10 to 12 seconds. I'm not sure exact number of time which it waits but for the acknowledgement. But if you have received, we should be able to immediately return because if you wait for your LLM to complete, it will take more than 30 to 40 seconds and GitHub will throw an error. So we're saying yes, we have received it.

I'll come back to you once I have something ready and then you can and then again we'll queuing the job and then it'll be coming and then it'll be publishing up there onto that pr. But the reason why we are doing that is a GitHub PR is eventually comes in it ingresses we have when verifies it queues the job into Redis and then it that PR goes to the orchestrator. The reason why we are doing that is again because it is so it asks for A I need to acknowledge fast. And sometimes a crashed orchestrator or any delayed orchestrator eventually may not be able to satisfy the GitHub condition because it expects the truly quick environment. Now where does this break at 10,000 PRS per minute.

Now Q which, which is basically the depth is basically outgrows the worker drain rate a single ARQ worker, basically the worker which you have, which will do the working job will become the quick bottleneck. So what you're doing we are using the modular monolith answer is extracting the webhook receiver as a standard stateless ingress service. And then the Orchestrator as a separate worker pool. So basically there's one receptionist who will keep on saying hey sure, give me the order. Hey sure, give me the order.

Hey sure, give me the order. And store it into the Redis job basically Redis database. And then it has a worker in the backend which you call this an orchestrator worker, which has a separate worker pool in the back end. So basically one by one this receptionist will keep giving to that, to that particular chef to create the food, to build the food, to cook the food. So just imagine if you just have one guy who's taking the orders and then building the same food, he will mess everything up.

So there's one receptionist, that's why we have a model monolithic architecture where it extracts the webhook receiver as a stateless ingress machine or service which stores something in Redis, which is arq. And then this gives one by one to this orchestrator which is a separate worker altogether. So what does this orchestrator does? So we are using an orchestration platform known as Langgraph. And I want to think about it a little bit.

Why are we using Langgraph as an orchestrator system? So basically the orchestration engine, it defines the workflow as a directed graph of notes which is functions of LLM calls and edges, which is what runs next. So the design pattern we are using here is. Here we have four specialists, right? For specialist who is reviewing my code.

So if you think about it, it's a quality security testing and basically this docs agent which is eventually doing from a different different perspective. So are the dependent upon each other? Obviously not. So what does it make sense to use basically a good design pattern, agentic design pattern for it is basically fan out all your agents parallelly so that everyone can work parallel. Because nobody is dependent upon each other.

My girl, okay, Nobody is dependent upon each other each other. So it eventually fans out and then then there's one accumulator which accumulates all this all the four workers. So if I show you the way it works is the orchestrator eventually have these four agents security quality test which is running in parallel and then this one aggregator which merges due dupe and scores and then routes it can either choose to go through. I mean it will first of all go to the confidence gate. If the confidence of review is good, it will post on GitHub.

Otherwise it will give human a loop. Right. Now the way to think about it is that there were two different, you know, orchestrator which we had to choose. The first one was the line graph and the second one was temporal. And again we are choosing this because it makes sense now, where, for example, if you want to use Lang graph where it runs, it runs in our python process itself.

It does not need an extra infrastructure which you are coding. A temporal is a separate server altogether. So that's why we are not going ahead with that. If you just want to use parallel, fan out because that's a selected design pattern. A Langgraph big is the first class via the send API.

Basically there's one behavioral pattern which is already there in Langgraph, but in temporal it is supported, but is heavier to express if you want to checkpoint for if you want to checkpoint to the same redis. Basically we want to continue to keep on checkpointing, but temporal actually have a very strong guarantees and built in for the checkpoint so that it can start where it failed. But Langraph you want to checkpoint to the same redis. We already run for the queue, so we want to keep on checkpointing there itself. The LLM integration is the.

I mean, Langgraph was built for these kind of things, so it has an amazing tooling integrations. But temporal is very generic, it is not LLMs specific. It is built for creating workflows, maturity is okay, Langgraph is new, thousands of this unproven. But temporal is very, very, very enterprise level. It is excellent at scale.

Operational cost is none beyond the app. But here you need to manage your own server, you need to understand your own workflow, shapes, etc. Etc. Etc. So for right now, for our worst case, we said, okay, let's go ahead and use Langraph.

But does that make temporal a bad candidate? Absolutely not. It depends upon, okay, for the MVP development, we may use Langraph, but if at some point of time we feel that we want to use temporal, we should, without thinking should be able to switch to temporal. So another major decision, our code should not be framework dependent. We should be creating a single safe intercept.

Basically, a discipline that makes it safe is a single abstract interface, which is workflow engine, which is an abstract class which has run the workflow, resume the workflow and get the state of that particular workflow. And then you're extending this abstract class to any implementation which you want, whether it can be land graph, temporal or any other orchestrator which you want to use. So let's say if the scale demands that you look at temporal after mvp, then you quickly extend your workflow engine. No code changes, because you have not built your entire thing on your language. I mean lang graph, you're just going to add a new Class of implementation of temporal which is going to run three different stuff which is run resume and get state which is going to extend these three methods and then we can use temporal at that point of time.

So the idea is is always and always make sure that you're trying to not make it framework agnostic. You're trying to make it framework agnostic. And as I said that this is what the architecture eventually follows. So this is what our orchestrator will eventually do. But we also need to do the retrieval design because we need to set it up with the right context.

We need to set it up with the right context. Now just wait for a second and I want to think about one thing is the design of the retrieval and whatever agents which you're doing it is not perfect. We might have a separate separate videos courses on how do you design state level agents? How do you make sure that you perfect the quality of the agents? Right now our goal is to create end to end system as an MVP so that you know but each problem.

When you say even a single security agent, what is the logic behind creating security agent? How are we managing state? How are we removing hallucination? How are you removing disambigation? How are you making sure that it is basically not taking a lot of time?

How are you making sure that whatever it is going to post is not unethical? It is basically ethical, right? So basically even at high level design you go at every single aspect. Right now we just designed very carefully about okay, this is how the ingest workflow should look like in real world. You end up sitting and then figure out okay, how does my ingress system work?

You design your ingress itself. Now once you go to the orchestrator you'll fuel spend a lot of your time in designing the logic of my security. How are you orchestrating trading? You may not choose to go with parallel. You may have a completely different type of architecture.

Depends man. I mean that's why you do literature review. So in one of the videos which I'm going to work on is basically I'm going to show you that how do you make reliable agents where we focus on a specific specific agents and write the logic as of now, this is good for the mvp. Similarly, we want a retrieval layer. Basically anytime our agent gets a PR we need to be able to quickly retrieve the PR difference, get the embedding of that particular PR difference.

Then because our code chunks is stored using ENN search and fts, we should be able to quickly merge it and type please make sure type cloud is providing this merge this, then give it to the specialist. So basically the PR Diffuderia Saint is getting the embedding and this embedding is compared to all the things codejunct which is available in the Tiger Cloud Vector store. We are using disk an in search and fts to be quickly able to identify the top K and merge those code chunks and then give the code chunks, the related code retrieved code chunks to the specialist. Every single specialist. Okay, now to think about another thing is your every single agent events which is basically for our observability.

That's why I say we also want to make sure that observability is up there, right there. So observability we are still we are using timescaledb Hyper Table is basically a trace viewer, audit trail and cost ledger. It should be able to tell us when the agent started and ended, what was the call, what was the cost to it, what was the. You know, where does that issue happen? Where the issue arised?

What is. You know, you can also keep on seeing a rising rejection rate per agent. For example, every all the agents are keep on rejecting your reviews which means not every PR review might be bad, right? So you know, you. You say that okay, something is going wrong, but suddenly you also see the developers keeps on disputing your AI agent reviews, which means that there can be another.

So we need the observability up there. We have already discussed in detail why we are needing observability but this agent's events will eventually help us in getting the right observability. So if you think about the whole system in general the MVP at least in the real world which you'll see is anytime the GitHub PR is posted, we're going to ingress very quickly using fast API which compares their signature with our unaltered key other potency keys and then says and then what I'm going to do, I'm going to post that PR and store that using in Redis using Archie and can queue that into a Redis job. And then the separate worker please understand this worker will go and tell to the GitHub peer I'm done. But then ARQ worker will pick one job at a time.

This orchestrator is Langraph. It will fan out four different security four different agents that will aggregate it will show his confidence. If the confidence is okay, it will go ahead and post the GitHub PR. If the confidence is not okay, it will put this human a loop so the human can go and approve and give comments and Then further improve and please understand the IT is going to be deployed. We are deploying on railway in general.

So your tiger cloud and timescale debug which is timescale DB is a one postgrid which means that you're going to retrieve the whenever the security because we need to set the retrieve design right. So basically the security agent needs the code differences. So we're going to use PR difference this architecture, embed the basic an insertion keywords, merge it and then fan out this specialist. Similarly we're going to use the PGVector Scale. This events hyper table will be going to be used by our observability dashboard so that we can observe it.

So basically we are not managing a different different databases. We're just managing a one managed postgres database itself which gives almost all our ability. For example in the economics and tokens in the phases we want to develop PR cost hourly what is the cost? Right? And then if it be.

If it is beyond our current cost, we may not go ahead with that. Right. So this is the whole architecture in general. Now if I think about it, if I want you to tell very quickly how I really want to build this architecture and I will tell you how I come up with these kind of phases. The first is the cognitive design.

I always suggest to the people ship the simplest version first. So we want to ship something and then we want to write the system architecture. Here we have taken a chance of saying okay, we're going to use modular architecture. Modular architecture and then the ADR says line graph over temporal. We always want that this particular confidence should go something like this.

So we want to store all our architecture decision records and then you have the backend and API. Frontend engineering is basically a simple dashboard so that somebody can see which of the reviews were sent for the HitLQ, the backend API. Basically our fast API which will validate our signature and idle potency keys. The workflow orchestrator which will fan of the four notes in parallel. Then your LLM and reasoning.

Basically model routing per agent so that it can route your agent and prompt prompt registry because you want to know which prompt was used and you also want to make sure see what happens. A lot of times we are also having a contact prompt of every agent right now we want to make. We want to keep an updating prompt or logic behind it, right? So for example for docs agent we don't want to use opus model, we want to use simple GLM 5.2 but for security we want fable model. So we should be able to route our models.

So we ultimately saying okay, that's one. Then we have a prompt registry which is basically one prompt was good in this and then we created another prompt so that should be also okay. This is the version 2 of the prompt, not override it so that we can always trace the decision back if something went wrong. Our memory architecture, the Piragon PG vector scale toolings basically scope. I mean basically we should be sandboxing our thing evaluation.

We should be for let's say that we are pushing the new prompts or new models or literally anything. So it should be able to at least pass our golden data set, which means that our new system should be at least better than the current system which is already deployed. That's what we mean by golden data set and regression gate blocks. Observability will be using OpenTelemetry which will land in the agents even hyper table. And then we'll be ultimately able to observe and trace everything which is handled the basically for security we'll be having RBC enforced and audit trail immutable.

For reliability we'll be continuously implementing the retries, the circuit breakers, the item potency verified under fault injections so that we have our reliable model. In general, we'll be having a separate session on how do you make reliable agents rather than reliable software engineering. And frankly speaking, we don't have the knowledge of security reliability. That's where the power of our Genesis kits comes. So Genesis kits is basically will give you all the required context to code with AI and give the judgment where it is required.

Okay? Your governance, your developer experience, your cicd, your human loop and you're continuously learning basically if the quorum continuous aggregates, if there's something needs to be changed right away. So these are the few phases which we'll be focusing throughout the project. And frankly speaking, I may do few of the phases and I will let you do most of the phases by yourself as well. Because the idea is if you know how to code and I will show you a generalizable version that you can literally code this entire project.

Right. So I'm going to set up a project architecture right now on the machine and then using Genesis Kit and then you will see the power of Genesis Kit right away. So let's get straight into the implementation. We will go ahead and we'll start the implementation now. What I eventually want is I have actually completed this project, coded this project.

So I will show you, maybe show you few phases, code in front of you, show you the way to Think about. Etc. Etc. Using Genesis Kit. Now what does Genesis Kit is?

Genesis Kit is basically. Basically the loop that prompt itself. You must have already heard about this something known as loop engineering. You must be already heard, I mean already heard about this harness engineering while coding with AI. Right.

This Genesis Kit is a framework developed by me and I would suggest you go to sometimes Anton Co and click on blocks and look at. I'll also put that in the description box below. There's one Genesis which eventually talks about the loop that prompt itself. And the structural reasoning is a missing piece of agentic AI. These two blocks are one of the most important blocks which you've introduced.

Talks about how to exactly work and code with AI in general and how do we give the state to our agent. So you will understand basically when I initialize this Genesis into the project, you will understand this. But what I want is if you can pause this video right now, read these two blocks and then come back, you'll truly understand what Genesis is in the behind. I want to show you a very quick thing and to show you that very quick thing we have something known as Agentic SWE Get. Now Agentic SWE get is one of the skills which will help us to take the domain knowledge of something like security engineering, reliability engineering, distributed systems.

It will also I will put up some screenshot of it how it looks in Obsidian. But it has an enormous number of concepts stored in the LLM wiki ways so that it can explore the right concepts at the right time. And it is natively integrated in Genesis Kit itself. Now Genesis Kit is basically that can actually help you to write plan every single thing for your project. What I'm going to do, I'm just going to copy this entire full onboarding which is basically you don't have to do anything.

You just copy your entire full onboarding right here. And then what you want to do is simply come up, come over here and you know, say you open Claude and I always like this dangerously. I mean permissions and of course feel free to do whatever you want and just click on enter. By the way, I am a big fan of Hermes, I'm a big fan of Codex, I'm a big fan of. I have all my agents running right now.

If I just toggle this up, you can see all the panels and my codecs, my Hermes, everyone is working right away. And I frankly don't want to. I mean I just want to show using plodding, but this works literally anywhere you want to use codecs, go ahead and use it you know, you Hermes, please go ahead and frankly speaking I'm the big fan of using Hermes. I'm not a big fan of using Claude, I mean a terminal or maybe even codecs but nowadays I'm trying to be very agnostic because a lot of my models are running on Hermes. I mean where I'm using, I mean at least the bedrock models, basically open source models there and most most importantly kind of model routing this particular CLAUDE specifically because I have a subscription.

So I mean it just gives a generous in because I'm recording this in around 3am in the night. So it gives a very generous and nobody's using so it just resets the limit by the morning. So that's I'm right now using it. But very rarely you will see me using Claude, I mean their terminal and not, not because I have any hate towards Claude. Amazing software, amazing stuff they're building.

I kind of end up using something where tokens are less. I'm very consistent and very, very conservative towards how much tokens I'm spending. I have one simple rule. If you have generated one token, tell me what was the output of that. If you cannot justify the output, I mean, dude, fuck you out.

That's my point of view. And sometimes Claude does misbehaves and a lot of time I can just use something like OPUS to orchestrate and then probably use Klein and Klein recently is giving a lot of free open Source models at $5 a month and actually get a lot of my work done using GLM 5.2 just having an orchestrator, right. So what I also do is sometimes have, let's say if Babel, you know right now Fable is available to all the people, but you can actually use Fable to be able to go ahead and as an orchestrator and sometimes and then use GLM 5.2 to write the code. So basically you're doing a lot of things from a cheap price. And I mean at least we'll have a different session on how do we optimize for tokens when working with AI encoding agents.

But right now Genesis Case does beautifully it giving you the right context, the right state so that you don't double write the code every time you start a new session. You will be able to use the same project again and again. You can start it, you can collaborate with the team without losing the context, without having the overload of the context, without your coding agent hallucinating, without coding agent doing whatever, without making you understand. We want to make sure that every tick which currently has a wall while working with AI. So first of all, there is, first of all, I want you to understand that if you think that working with AI is like wipe coding.

No, this is not wipe coding. Here you will take a lot of time in system design itself and then create few stuff and then eventually come to the cloud itself. So it's not by coding, it's basically how do you harness, how do you design loops? How do you design debug loops, how do you design research loops? How do you harness your model?

What are the guardrails? Are you having? What is the token budget? How are you routing your models? Basically, how are you verifying your models?

For example, one of the things which Genesis has is a lot of times your model itself writes its own test cases, but here it will not write its own case test case. In general, it will verify. Because why it's like, why does not write such test cases, which is just one feature of Genesis Kit, is that it's like verifying or writing the exam paper for your own and then verifying your own exam paper. You'll automatically write those exam questions which you're already aware of. Because the teacher creates it.

Because teacher does not know the reasoning, but knows, okay, this is this, this, this is the input and that is the output. This is the test cases which must agree. So what I'm going to do, I'm going to paste my entire prompt from the GitHub repository. And by the way guys, this is an open source repository. Feel free to go ahead, contribute as much as possible.

We want to make this popular. We want, we want to make this true state of working with any coding agents, no matter what coding agent you're trying to use. Glm, Kimi, any cheap model, any expensive model, it should be able to use that and feel free to star it and feel free to fork it and then create a new pull request. And we are happily accepting pull requests right now. So if you go ahead and shift it, but before even that, I just want to make sure that what model I'm using and I'm 100% sure that it's using Fable and yeah, it's using Fabul.

So I technically going to use Opus and what I'm going to do, I'm going to paste this and I'm going to do a very important aspect over here, I'm going to say can you please make sure that you are not using opus, you're just using OPUS as an orchestrator or sometime planner. Whenever required, spawn haiku or Sonnet 5 agent to be able to do this task, never spend a lot of your tokens on this itself. So always spawn Haiku. Always spawn Sonnet 5 to do this particular task, even for the coding agents, I mean materialize over the time. So go ahead, initialize Genesis right here in this project.

Always use Sonnet to even do anything of that kind to not dump all your opus tokens here. Now it's an. It's, it's, it's. It's a very important thing which I told is because it's the mechanical work and, and you'll notice throughout the way that how I'm using these tricks and we, but, but, but we'll have a separate session on, you know, working with coding agents in general. So I really don't want to.

I don't like dynamic workflows because it does not. I mean it is not of my favor in general because dynamic workflows sometimes dance in parallel. But most of the times our projects sometimes are very dependent with each other and that's why I'm very particularly against using workflows. But it's a beautiful, beautiful one if you're truly sure of that. None of these two agents collaborate with each other.

So let's wait for some time it completes. It. All right, so where should the Genesis spine go in Genesis? Because it is our memory of whatever we'll do in the project. I'm going to do one thing.

I'm going to say, okay, I'm going to use this folder itself and I'm building an AIPR review agent and I will slightly add a note and. Sure. Okay, let's just go ahead and submit the answers. All right, so it is a. It.

It has spawned some agent. I'm not sure which agent to say it has spawned. Oh well, I think it has spawned solid. All right, so it is doing something and not something. I'm very sure of what it is doing.

So it is basically making sure that do not write over an existing. If it prints no Genesis yet, proceed and make a GitHub repository. Explore the kit, run the graphizer and then understand and then run the gates. Now you'll see how the, your particularly your Genesis will work beautifully right now. Alright, so as you can see, this, this HTML file is our teaching file, the Notes file.

You can always go there and see it. So basically you can see what it has created. The first file this created is. I mean a lot of files then and the first file, I mean the things which is very important for us. You should see over here something known as done HTML and implementation Notes HTML Done HTML says that this is what done looks like.

Your agent is not allowed to touch this done HTML. You're only allowed to touch the done HTML which is basically says okay, this is something which no agent will touch. This is what you call is done. Because a lot of time your AI coding agent can divert and change the goal as per its convenience. So you're not letting anyone touch this down HTML.

So that's why here it says scope. What kind of work is this? Is it a build from scratch and there's an existing code or there's some incident we are trying to fix? There's a new build completely so we should, we should just say a new build and then is this distributed? What's the runtime shape?

Basically reacting to get a possibly with a queue. Real distributed multiple services. Okay, so I'm going to go ahead and having the webhook service because we said that a long running Service reacting to GitHub webhooks basically PR open and updated possibly with a queue instead of having to give a lot of answers because I think that we have already designed our system so we don't want to give a lot of the answers. I'm going to say a lot of your answers chat about this. A lot of your diagnostic cognitive answer can be found here.

However, if you're restarting your project, this actually gives you a good way to think about. So this is. We have already designed the system in general and that's what we spend a lot of it a lot of our time. So it that the. The gate zero itself is to making sure that your done HTML gets filled with exactly what you want to eventually see.

And again see the problem with this. I mean what I don't like with Claude that one time I stated that can you please go ahead and always pawn any coding agent and not just opus but basically Sonnet 5 to be able to do a lot of its work. But yeah, as you can see. Oh okay. Finally.

Finally. Okay, so I think it has extract design Doc for Genesis. So it is able to use Sonnet to be able to design Doc and not using OPUS for a lot of talking tokens. And I love it frankly speaking somewhat. Claude.

Claude. Guys, does it. Guys. I'm not saying that Claude is bad, I'm just. I'm just saying they're, they're.

They're just not okay in terms of what they do with me. So. Right. So by the way you notice right now I just pasted my design pattern. That's why the design is very important.

Because cognitive job is one of the most important job and that cannot be templatized because there are a lot of business decisions which is being involved. It. All right, let me read the design doc and I will design the sonic and pull out everything relevant to the rituals. Basically there are five gates that can actually write your cognitive job. Okay.

There's one specialist LLM agents, the one aggregator, the LM as a judge the OpenAI text embedding three large models. Then there's new build and there's runtime shape which is basically distributed. As I said that we want to use the fast API ingress store it in redis arqq langraff worker will be spawn out which will be four separate nodes. And I'm just verifying in this one aggregator which aggregates HTL and post back to GitHub we're having a next year stashboard. We are having timescale tiger data as a spine for our storing our several types of memory.

And we're going to deploy on railway. Amazing. The trust boundary is that we want to have webhooks which is matching with each other the PR difference which is basically untrusted and prompt injection. So we are also looking for making sure that none of the security stuff is happening. The LLM outage should be always grounded, should be always confident and rational and followed by our human loop agent.

So we are done with the design pretty much great. So how should I build a 20 design into Genesis? Each milestone needs one exact. As I said, one of the beautiful part of the genesis it is a lot of times your coding agent actually produces something and says okay, it should work but for this we want to make sure that it has some sort of harness that it will call done so which means after every phase we want to make sure that it is going towards and basically through our, you know, the style which we want which we. That's.

That's why it forces it to run a specific demo command that proves it works. For example, let's say that there is one, you know, let's say webhook ingress. Now webhook ingress is basically the demo after completing this milestone would be curl signed versus unsigned. Okay. HMSE rejects idle potency drops.

So this is basically the demo. Then the langgraph demo is trigger run four nodes in parallel kills the worker resumes from the checkpoint. So basically run the rows in parallel kill the worker and then see if it reviews from the checkpoint as well. So that is another one specialist press aggregator Summit test PR different one merged prestige Rag retrieval hybrid query on choke code chunks and all that. Then observability fault and checked and all that thing which is again an amazing.

So now it says for every basis we want to have each notes. So I really like to do something of this kind because I mean we should be able to. So let me say. I mean we should be able to. I mean so tell me whatever you need from my side, like something you need my keys from GitHub, webhook and probably tiger cloud keys and whatever you need.

And by the way also give me step by step to get and retrieve those credentials and then to put all we put in the dot ENV file and. And probably we can skip. I mean I want to do individually and have the demo commands individually. Because see we. We may.

We may choose to. So do not spend a lot of time in front end. It should be very very simple. Do not spend a lot of tokens in that very simple. It's just that I want to see there.

We should prioritize our data basically tiger data a little bit so that we can sort the tiger infra rather than that coming out of some time so that we can see what things are being stored. So let's plan in order where the parts which is not dependent for example Tiger infra then we can eventually talk about. So even before tiger infra we can build that particular webhook ingestion which is ingress service and then we can build this tiger infra so that all our infrastructure is sorted the first place. And then we can go ahead and having this orchestration, the multi agent and all that thing. While I want you to make sure that you put it somewhere so that we can eventually get started on that front.

And feel free to tell me whatever you need from my side. Give me step by step way to get me up there. I mean get to. To have you give me the credentials up there. For example, you might need this Redis.

You know, Redis Spanish database. So you know, you tell me. I mean where. Which. Which particular tool should I use which.

Which particular platform should I use to be able to give you to. To. To. To basically store the jobs. And of course I want free stuff right now because of course because I'm not developing this production.

I mean I'm not. I'm not scaling going to scale. See, it is going to do every single thing. Whatever we have said by the way, it has not started. It is still confirming.

It. Okay, let me quickly go ahead and see the milestones what it has created. So as per our Plan the webhook okay, that's fair. Then Tiger infrastructure. So the demo command is something like this that it should be able to figure out the tiger and send webhook query C spanrows which is basically every event spine every action pen only agent events then orchestration which is trigger run four nodes kill worker resumes from the checkpoint.

And we want to make sure that the external dependency over here is basically taking stuff from the redis itself then specialists. So basically we want the OpenAI to be able to do a lot of things which is basically open AI keys, the rag retrieval basically our tiger plus OpenAI where you're retrieving something then hitl gate and post to GitHub which is the test pr the reliability then minimal dashboard to see a lot of it and then the evaluation. So pretty much okay, now you'll drop this. So GitHub it needs my GitHub webhook secret which is quite amazing. For M1 which is free and self generated I should be able to give you.

Then it has Tiger Cloud for M2 which is the free tier which means you already have the Tiger MC connected in this session. So I can provision the database directly. When we hit I'll run then it returns the connection string. But either do it by sign up a free ad and create a service copy. So I have already given you the step by step plan so you should be able to quickly get your tiger database because since I already have it, you don't need to eventually then redis which is basically up stash serverless redis free tier perfect for AIQ sign up which is basically when you use up stash.com service list redis and then we're going to put that into redis URL which is total of 30mb which you get for absolutely free.

I'm not sure in this maybe you pick a region, you create a database then you have a connection URL from the up stash itself. Then for OpenAI for pay as you go pennies at this scale. So basically you want to use API key and I'm going to use bedrock whatever I have into my current system GitHub tokens to post reviews for M7 which is GitHub which is basically I need to be able to post it back, right? So I should have something. So pretty much okay.

And how do you want the tight cloud provisioned when reaching. Okay, so basically they have MCP up there so you can actually use Tiger mcp. So basically if you go to Tiger data MCP and I'm going To show you right away and they have an amazing MCP by. Oh, okay. Okay Tiger.

Alright so feel free to go ahead and first of all create an account using the link given in the below and then go ahead and try to authenticate and then it should be able to install your MCP from there. Now MCP can actually do a lot of things. I mean it can actually pull up all your things and run a lot of actions on your behalf. So sure. I mean yes, we have a provision so sure right now that's okay, I can run it.

But basically feel free to go ahead and have your same thing whatever it asks. By the way the best thing is that you can actually do whatever has your computer PC Right now I have this beautiful Mac and it does have. I have a lot of background information so I didn't really appreciate not going that. But feel free to go ahead and let's say that you go to this something known as console, the timescale dot create service and then you can just go ahead and go to the Tiger cloud and login. So let's say if I continue with Google and by the way intelligent programmer123 at the rate gmail.com is 6 years old email id amazing.

So you go over here and then you can quickly go to create a service. So I already have it so feel free to go and create the service right up here and you select maybe the shed, you have that and then you continue and then you name your service. Then let's say I'm going to re test and then create a service. Now once you create a service you can actually go ahead and your service is going to being deployed and I'm not sure if that. But this is your connection string which you ultimately have to give which is ultimately it is asking is Tiger database URL which is trying to create.

So sure use Tiger MCP or you can find basically the connection string in Tiger in an AI PR review. I'm gonna. Because I don't want to give it once again Agent Tiger data folder and desktop as well string connection string is needed. By the way guys, I use an open source, I mean model for dictation. I don't use Whisper.

I don't know, I just don't like the attitude of the founders so and, and by the way I'm gonna. Am I going to pay you even a single penny to dictate? Are you serious? I mean to dictate? The whole model is flawed man.

I mean the world is going into a very weird space. I don't like whatever is going on. I mean literally there are a lot of free models, dude. Pay for something which provides value to dictate. Do I really?

Okay, I don't want to go there but amazing now. Okay, sorry guys. By the way, Tanya is an amazing person. I like whisper flow but I'm never gonna pay for dictating. However, if at any point of time if you can convert my dictating to an actual agenting workflows than I take my credit card, man.

But no, I mean you have raised hundreds of millions of dollars. You're an amazing entrepreneur, amazing work. You have literally started this dictation as a space and you own it. So a huge congratulations to Thane as well. Okay, all right, so it says no date.

I'll use Tiger mcp. By the way, you have still not started the code. We're still here. Okay, so it is asking me to you know, write GitHub webhook secret and frankly speaking what I want is I don't want to you give another one so you can actually go ahead and create your own GitHub webhook. Feel free to search.

So basically that's a secret token and I don't want to create in front of you guys. So I'm going to say okay, I have already created it somewhere so let it complete and I will tell that you can actually use from Env file of aiprview agent. Yeah, let's see what it is trying to create now. Plan md. Okay, perfect.

Let's see what it does, not what it does. Every file has to be approved. I tame my coding agents. They're very sweet. You see, implementation has gone very easier.

However, if you don't have the harnesses, you're going to create a worse code of your life. If you have the harnesses, right? You're going to create the best code of your life. Right. I'll pause the video and as soon as it is done, I'll come back to you.

Alright guys, so here's what it ended up creating. Now basically what I want is I want to look at what it did after it initialized the Genesis kit. It eventually came to this point and said very, very, very, very, very, very. So let's quickly see what done the HTML looks like. By the way, I will delete my connection string.

So don't try to copy that. So this is a locked spec. What is the cognitive job? What is the input? What is the autonomy level?

How are you going to tolerate the failure? This is what we are building in general. Then what is the dependency which means dependency direction. Every outbound call has a timeout LM output validated against threat model return one verify pass by a separate agent for every milestone or every invariant should not be this is the verifier which verifies then for every what is my phase? What is the demo command that it works?

How are we going to use the loop? For example, there are three types of loop build loop and debug loop and research loop. So we're going to use the loop and then try to create I'll come to the loop in general then skills which we are loading per for example, for the first milestone where you load loading data system engineering from Agentix SW Master Model Architecture and Coding Orchestrator then M3 where you're using security engineering and distributed systems etc. Etc. So it is using the right skills at the right time.

So done. HTML figures out what is the locked job. Then your genesis is the one which is which got unfolded. And then you have this implementation notes. Now what this implementation notes is as soon as see a lot of times your agent sometimes creates something new even if it is already there.

So it checks that what is live right now, which means what is the now in the flight? What is your active loop? What is your current milestone, which phase you are in? Is there any blocker? So anytime it creates something new to look at this implementation notes and see that nothing is being contradictory.

If things is already there, we'll go ahead and do it. So this implementation notes is like a live state which is given to your machine or your system so that you can continuously have a context of what it is building. So even your next guy can actually work and work on this. Then it has this context graph. Context graph is.

Let's say that you have one code, right? Which is what are your invariants. The invariance can be every GitHub book payload passes through a signature. Every webhook delivery is deduplicated via its own delivery at idle potency key. Every specialist finding carries confidence and rationale.

These are the invariants which is my business decisions, which is my rules. Non non negotiables. I mean if you must. If you know about what are invariants into a system. Invariants are something which you cannot violate.

So your context graph will contain all these invariants. And then then this. This is something which cannot be violated. Also your code is given a graph structure so that if something changes one point what other test cases must be written so that other does does not get affected. So that is your graph.

Then you have index basically if you look at this index.md it contains basically sources, the concepts, how it works, the things which the system has, etc, etc. It acts like a documentation to your project. So this is given by Andres Karpathi and amazingly I am using this into my Genesis kit as well. Then your plan MD is a very, very detailed plan. Basically what is your approach?

For example, we brainstormed a little bit. The approach was modular Monolith plus Langgraph plus Tigerspine so one fast API holding webhook interest. This is what we ultimately thought of. Then approach B can be microservices per agent. So there was recently Tariq also talked about the guy from Claude Code.

He said that if you do not know what you do, what are the approaches to start you won't be able to do. Only so for every milestone can we figure out our unknowns, something which we are not aware about. So there might be multiple approaches. First of all, we go ahead with a simple approach for this kind of task, which means choose modular model IT langraph, orchestration swap, Tiger cloud build order prioritizes non dependent pieces. First webhook ingress, then Tiger data and then ultimately goes ahead.

So for every milestone it has what is the outcome, what is the phase, what is the files and bound it touches, what is the demo command, what is the success criteria, what loops it runs, what skills it adds. Basically this Genesis kit itself gives a lot of skills which is needed, the external dependency which is which it is dependent upon and what is the token budget we are giving to complete this particular milestone. You see an amazing structure and how it is done and several other milestones out here itself. Now if you go ahead and look at the loops md. Loops MD is an instruction is then how AIPR review agent eventually gets built.

So basically before any existence, it runs G0. So it looks at that G0 procedure, which means they did pick the candidate pages by the name against the milestone. Now, so which means look at this state. Basically whatever we are developing, we want to know what we are developing because we don't have the context yet. So it picks the candidate pages by the name and against the milestone noun so that it can know that what it is trying to do, it will drill into those existing pages.

Initially they may not be existing pages, but let's say if you come back to M1 after building everything to make it better. So you ultimately will come and drill into that. Then you search. If you are starting something in milestone, there can be multiple things, right? You must have closed your Internet, something must have stopped.

So you look at implementation notes to see if there's anything rolling source of truth what's already live. You confirm that the system does not already exist. Once that all these checks are done, you go ahead and write a G0 checkpoint. G0 checkpoint is basically current MD. Your current MD will contain all the checkpoints.

Whatever is happening then G0 verdict says okay, it's unbuilt. Okay, if it is unbuilt, go ahead and build it. So every milestone will run this. Then how it is operated. It is cheap by default.

We are using Haiku or sonnet model and opus for the expensive. I mean lot of difficult tasks. We have the coding orchestrator to be able to and then we have several agent tickets W kit and cognitive skills. For example we have Detective which actually helps us in debug verify which actually helps in check output. Blueprint is basically design before you build.

Scout is basically explore option to solve a problem Council and mirror which is adversarial self review. Ghost is basically shadow and carry force that is basically predicting the failures. So these are the skills which I actually made for myself. And I have completely open sourced it and it works amazingly well. Then for EV, there are five gates after G0 skill which means did we load the right skills, the router which is needed, the loop required skill.

Did we log the skills into the checkpoint? Did this iterably this particular task which we did into that particular iteration. Basically to complete that loop there can be multiple iteration. Did that particular iteration actually measurably move the milestone forward? And then were all everything in budget.

Did we verify the quality and then did an independent checker and computed based on the demo command. Okay, now so the way your loop will work is the five loops over here. The let's say for the milestone have multiple smaller tasks. So it will run a loop based on the milestone. It will start a loop.

While not milestone is done. Then the iterations are less than 10. First of all, you create a micro plan for the particular thing. You create a milestone specific plan. You load the skills, you read all the wiki pages.

You checkpoint the initialization of this milestone. Then you produce a micro plan. You write to the checkpoint. Then you create a loop. See while not milestone done iterations less than one load the skills, load the next phase.

See if the phase needs research. If the phase needs research, call the research loop. But if it is not edited edit files, run the test. See if it actually helps you to progress. If failed, run the debug loop and keep on continuing until you eventually get your milestone done.

Once a loop is completed, then you spawn a separate verifier, a separate coding agent that can actually verify if it is correct. And if it is correct, feel free to say, okay, milestone is done. But if it's not Correct, spawn the L2 debug retry just once. If it is not update in the checkpoint and append the progress to your plan. And your basically plan md and implementation nodes.HTML similarly, it will call this debug loop.

So everything, even the debug debug comes in the loop. So if it calls the research loop, this research loop will keep on looping until this task is completed. Right? So you can go ahead and read more about my program genesis case in detail there. But this is what exactly which will happen now.

Once you have everything ready, you have to go ahead and put this. This thing into your ENV example. You can ask whoever your coding agent you're working with to help you. If you do not know where to get all of this. Great, I've already added it.

What I'm going to do, I'm going to say let's. Okay, let's do one thing. Let's say what happened is. Over. Everything over.

What shall we do? Let's say I open a new cloud session. Okay, Fair enough. Let's open a new cloud session. All right, So what shall we do?

We don't know. We don't know what we have done. The beautiful part is that look at the kickoff MD it. Copy this. Not even this.

Anytime, Literally anytime. Kick off whatever you're doing. Okay. Okay. I think it did.

Okay, great. So if the agents are named not there, it will go to required files. Now, if you have already have something so it looks for something which is already there. It. All right, let's see.

By the way, this is completely new session, no context, no state, nothing. Great. It create the checkpoint M1MD verdict is unbuilt. So it says definitely yes. Your M1 is not yet built.

It says starting L1 build for M1. I'll scale for the project, write the four modules with the tests. So now it will start your M1. It should. The problem is that it should ask.

I have a habit. You should always ask me before you do anything. But yeah, that was the point is always start from where you left. So it is going to start off with this and let's go to backend and let's go to webhook Receiver and then you have. Okay, so I'll do one thing.

Let's say that okay, how we'll understand the code. So Genesis kits gives you an amazing way to understand your code as well. So there's a new thing which I added and what it does. Let me, let me, let me, let me show you where it is. Where it is, where it is, where it is.

Where it is. Where it is, where it is, where it is. Yes, the kickoff. Which means that you can actually put kickoff interview to. I mean to interview you before you develop a particular milestone.

So before you get a milestone, before it even writes a milestone, you can actually write, you know, kickoff interview in general. Okay. The kickoff interview is basically making sure whatever you're going to develop it produces the right thing. And then you can also call them quiz me basically. Quiz me is basically helping you to verify whatever the core it has created that you truly understand.

So if you want to truly understand and I suggest everyone to run. So instead of saying okay, it looks good, now it also runs the thing, but you also say, can you quiz me? Alright, so let me quickly show you something. Genesis current.md m1.md now if you see over here your M1 progress ingestion progress wiki page is red Implementation notes Nothing. It was the verdict was G0.

That is the first gate in iteration number one. This is what it did then verify. The next is the verification now. Now spawning an independent L4 verify agent with fresh context. It only gets the goal success criteria and invariance not our builder trail.

Why? Because we need a separate checker to test. That's why a new agent is invoked. Basically the word the work of this. You're an independent code reviewer for a project called not write this code verified cold adverse.

Do not assume the build logs claim such milestone goal success criteria is this. What are the invariants? Something which is in negotiable non negotiable towards the what to do to test it. That's it. And then it eventually comes back and then it will say okay, the M1 is technically done.

Now the beautiful part of this kind of system is basically telling you that okay, this is what needs to be done. So what I'm going to do, I'm going to quickly. Show you. By the way, I'm definitely not going to work on all phases. I'm going to work on maybe two or three milestones.

And ultimately because see a lot of times you can only complete the milestones by yourself. My idea was to teach you through the system design. So I'm going to develop a couple of more milestones and then let you develop built on top of it. Because if you just ask me, it's damn easy after that because the major part you have already done if you have the right harnesses set up. By the way, you won't get a single thing if you don't understand how Genesis works.

So please go ahead and watch the blog of Genesis. But I kind of love it. So see the way it has created. It has created that JSON file you can see over here, which is basically sample print and it is reviewing somewhere the verify approved with one major malformed word validity sign crashes where instead of clean. Let me fix that quickly before now.

If you see that your current agent said it's good, but your verifier said something was wrong, right? Then it comes back, the debug loop starts this fixes it again. Verifier will check it. It also adds a regression test. What is the regression test?

Regression test is basically if something goes wrong. Okay? If something goes wrong, did other things also gets very. If if I change something that some it did other things gets verified. Then you have this L4 which quizzes you as per the protocol to make sure that you understand every single thing so that you understand your milestone one so that you cannot claim that it was not your decision.

Why does receive check before HMSE rather than giving the principal should get all pro? I mean basically given the principal that verification should get all processing. So it's saying It's a fixed 400 response regardless, so it leaks nothing useful to an attacker. You think signature verification should happen first regardless and this ordering should be fixed and not sure and skip which is an amazing question and I wanted to answer it. Basically it's asking of a very very interesting question the water.

So okay, the question is very very simple is how? What do you do that pause this video and tell me. So basically for the first one is obviously that you have a fixed first option which should be correct is because the principle was verify before processing. Because we don't want to do any processing before verification. If you do not know that if it is coming from the right source, it eventually contradicts one of our invariant.

And that's why we want to say that's why we should be immediately returning the bad request. If it does not matches with our signature, our verification does not occur. So verification first and then processing later. Then we have another edge case before the fix just applied malformed but validity signed JSON caused a raw 5 is 400 the right response or should it be something else? You have two seconds.

400 response is correct. The Reason why we are saying that is because once your HMSE signature has been successfully verified, you are going to already establish that the request came from someone who passes who possesses the shared secret or somewhere from legitimately from GitHub. So at this point the request is authenticated. If the authenticated request does not contains your JSON which is malformed, that's a client error. The returning bad request accurately communicates that request body is invalid.

So the ideal flow is that we verify the HMAC, parse the JSON and then parse and then returns 400 bad requests and continue processing. If the parsing basically your verification and processing succeeds. Now you have another question is also by the way that and also by the way you can actually chat with. If you want to understand that's why these questions are there. You're not intended to answer it right away.

Go ahead and chat about this, right? It will tell you okay, why are we putting 400? Why are you not putting 500? You know, what are the other benefits of it? What are security benefits of it?

What are the practical. For example, over here the practical benefits is very amazing. For example, it will prevent your false alarms and server monitoring because the mal informed input is not always a server fault. It might be somebody trying to attack you or it can actually help you make debugging webhook integrations very easier. And also aligns with the HTTP standards like semantics which eventually see.

So ultimately these are the. I mean for you the learning opportunity and more importantly, why are you doing that? What is your approach behind that? And truly understand the code which which you have written. Then you have another question which is change impact that input.

The ident job router is in memory only. It does not survive a process restart and it due to strictly is that acceptable for M1 given M4 will replace it with redis back due to which is probably correct. Which means the M1 goal is right now is to establish the web hook into ingress contract. So what it does it authenticates the requests, validate the required headers and payload that is coming from the GitHub route the events and prevent duplicate processing within a single process lifetime. So what it does if you eventually go to your you know the file, click on your most favorite backend and click on webhook and basically you click on App py.

You can see that what is going to happen is basically that it is the webhook anything which will come over here. So it will come at this. Your request will eventually come come over here. So if it is not A pull request is going to return an error. But if it is a, you verify the signature.

If the GitHub request is not, you say it is, it's not something which I want. You get the JSON, you parse the pull request basically into the required format and then you queue that into a job router. Right now it's all right. In M4 will queue that into the. You know, our most favorite redis.

Alright, so over here, if you look at your router and what is router does is the real queue. Basically it should queue it somewhere. The PR should be queuing it somewhere. That real arrives in our redis queue right now. But for right now to get the ingress contract and to end without a broker dependency as per our invariance, it eventually ended up creating a very simple queuing mechanism itself.

Right now, very local stuff, which means something. Until we create our own redis queuing mechanism and it is making sure that is this pull request repetitive or not, which means it is checking for is it duplicate or not? If it is duplicate, it is creating an alarm. So what it is technically saying, hey, it might be that sometimes you may, your GitHub webhook might retry or anything of that kind. So it should also check for duplicates, which means it prevents the duplicates right now.

So right now it's a temporary thing. We'll ultimately have our durable share due to state via redis. Right, I'm going to select the some of the answers and I'm going to go ahead. By the way guys, please always run, please always use terminal. Guys, do not use desktop Cloud Desktop Codex Desktop.

Alright, so it says Your verdict is M1 approve which means it also approved us. And then as an exit protocol, current MD plan MD and progress log and implementation not score is going to be updated everywhere. So great. Even the PlannerMD is going to be updated. Sorry, hallucination it.

Great. Refresh what is built right now, which is webhook ingress webhook ingress webhook ingrock parse routing. Right now it is queuing the job. Right now it is stub. Basically we have just created something.

It is in milestone four. It is going to store in redis. Okay, Known gaps, which means what, what are the gaps right now? Tiger Cloud provisioning, which is basically service pro zip provisioning. An M1 follow which is job queue is in the process tub.

It needs to be replaced with redis a ARQ the session log. That is it. If you understand, you understand your, your, your, your, your guy can come and Then they can start working right here. See what, what we did, we harnessed it. It's one of the Beautiful, beautiful.

And I love it when it does. And, and I love my Genesis kit, guys. And please feel free to make more pr. And I keep on, I'll also keep on improving and amazing. Okay, so let's go, let's, let's, let's, let's try M2 and then I will let you to do a lot of the work by yourself after that.

Right? So let's go, let's say let's go to M2. Tell me what you'll do first. What you will do first. Oh, let's go to M2, man.

I mean basically what it's trying to do is I have opened a new terminal agent and I want. So basically our work was that. Let me, let me, let me show you. And, and what happened that my previous service was not working and that's what I saw. I actually went for the coffee and then it said okay, all right, let's see if something needs to be done.

So it goes ahead. It eventually say this is the service which is already wired. Your test service is basically, basically created in front of you guys. But my previous Surface was paused because of obvious payment issues. So that's why, so that's why I told okay, I can create a new account and you can actually create a new account and you don't need to, you know, if you will.

Most of the things are actually free, but you can create a new account which will get you thousand dollars in credits and you can actually use it to create production environment. So go to the target cloud account using this and then use my link down below and then you can actually go ahead and re authenticate. So you'll be coming out here which is services. Click on CLI and mcp. Install any one of it.

If you're Windows, if you're Linux, figure out by yourself and click on up here and install the script. Now click on Tiger Auth login. So basically that's what I did over here is I installed it, I actually deleted it. And just to show you guys and Tiger Auth login and then you can just go ahead and authorize your Tiger CLI because this will give you an MCP so that your CLAUDE can do a lot of its tasks. It says all set up and then it says let's truly test the services.

Now I'm going to say okay, Tiger service list, probably speaking, there's no services. Right, Right. So okay, so I'm gonna say Hey, I okay, let's I reon I re authenticated new account is created. Feel free to use their CLI/MCP to create a service of our choice. Same as previous one and go ahead.

Building M2 as you can see it is in G0 which is pre existing flight. So let's see what it does. I'm going to close this loop. I'm going to just wait. It called the Tiger which means mcp.

It is calling the Tiger once again. It is creating a dedicated CPU as you can see. Time series AI add on such as timescale DB plus vector and later. We need the PG vector scale as well as per our design principles. Oops, guys, this is not needed.

It really shown in front of you guys and you really think that I'm gonna have it? I'm literally gonna delete every services. You can't even see anything. I want to delete that account itself because I no more need that Anyways. Whatever is shown, it's fine.

It. Still at G0. Okay, so it says go to a checkpoint CM2 the G0. This is the things which is read. This is the code break code grip it did.

This is account state. This is what on the new service, the table it has created this decision that binds us basically immutable the agents events is append only and immutable. You cannot change it because we should trace it and verdict is unbuilt. So it creates. It goes towards a loop.

So Loop L1 build is basically the loop. So let's see what it does. Let's apply the migration directly and against the live Service. So your G0 is completed. Now it is going applying migration to Tiger service itself.

And we'll see. I mean let's apply directly to the Tiger MCP against the live service. And that's what it is doing technically. So if I literally show you what's happening is this memory, something will be created here itself. And the whole batch rules are not supported in hyper table switching to a trigger based append only.

So basically if you see the migrations basically the scripts and the migration switch is technically doing. So it's creating all the memory shapes and tables which is needed. As per our design principle. It. Every time I know I'm opening Twitter, something is coming up there in live.

Nice. Both update and delete are hard rejected. Perfect. Amazing. You see how beautiful it is doing env4 which is a context driver.

Genesis enforced at the database level, not just the convention. Now writing that which is and smoke test that exercise it against the live Service. As you can see that every invariance is being followed. That is one of the beautiful part of what we have designed. It is getting the connections going to record the event.

And I love it when it follows these things then. Very dumb though. I mean these are things which you should identify prior itself. All right, let's see how it goes. And here you see the tests.

By the way, As you can see, the requirement Txt is continuously being added. Agent events cannot be deleted as per our invariant. So test data even will it will live in those agents events so that nothing is being deleted. Now writing the Checkpoint and running L4 Verify. Now L4 our different verifier is going to be spawned to test whether Tiger data infra is working or not.

So basically something has been done. It says all the gates has been passed, decisions which has been taken. And then running the M4. I love it. Okay, how easy it gets once you have everything harnesses and everything done.

But yeah, you have to keep on. You know, it will keep on verifying whether you truly understand everything or not. Okay, so L4 verify tiger data. So you can see out here is basically your independent for a project call. We did not build this verified cold and we do not trust Tiger provision extension enable agent events hyper table plus continuous aggregates press relational tables and finally records hitl feedback created via migration.

The success criteria is what are the invariants which is at risk. The non negotiables now spawning an independent your tiger will be executing in SQL query. Okay, What is the next phase? So let's open plan md. Okay, see we are living under the budget tokens as well.

M3 is right. Append money rows to agent events for every action. Webhooking grass basically. So basically every for every action. Every action for a delivery is curable in agent event.

So basically for the observability we need to be able to quickly pull all the traces which is correct. And then you have this orchestrator which eventually fans out and fans in. And then you need an external dependencies here and then rag retrieval HRTL gate post to get up reliability and economics and minimal dashboard. Keep tiny and roll over investors. See the power of our verifier.

There's one real gap. So be completing with this M2 and you should keep on continuing and it is going to work. So you'll be getting exactly this file and you should continue continuing with M3, M4, M5 and all the way down to the M until you get your GitHub token up there live. Okay, see here it Asks the questions now, but let's say that if. Let's say reject agent were uses before update rather than a postgres rule, why was a trigger chosen?

And what property of a hyper tables makes rules unusable? Here, take a second. I'm gonna get a. Get a water and then come back. Okay guys, I'm back.

So tell me more about it. I mean answer, answer the first question itself. So what does the first question answer should be? It should be the first option itself, which means is based on our design rational the rule in the PostgreSQL they are implemented at the query rewrite stage, which means the timescale db Hyper tables internally distribute data across many chunk tables, but they never ever rewrite rules, which means attempting to create one result in an error along the lines of. So basically it might create different chunk tables, but hyper table will never ever support this particular rule.

So let's click the first one now if you understand how by this design, that's, that's how we'll start to understand the code. If, if you don't understand, don't worry chat about this going deeper. Given that I've already worked on the project, I'll be able to make it much, much more easier. But why did we choose hyper tables? See, hyper tables will be rejected outright if it tries to write because fundamentally rules in postgres are implemented at the query rewrite stage.

So hyper tables may distribute data, but can never rewrite the rules outright. You can tell. Can you tell me some example of what you're talking about? It should be able to tell you something. Then edge case, which means if a caller runs truncuate agent events instead of delete, what happens?

Does the delete trigger fire? Guys, This was the error which we got after the verifier and most of you would not understand what it did and most of you would say okay, it has fixed. No, no, you have to understand what happened. That's why it asked this question. So basically, trunkuate is not implemented as series of delete operations.

So it is a distinct SQL command with its own execution path. So anything which is like before delete, after delete row level triggers do not fire and trunctuate to intercept and trunk to wait. You should be having this before trunk to wait or after truncuate right there. So our first is no needed a separate trigger itself. It should not be used instead of delete.

That was the edge case which we found, I mean when running from the separate verifier. So there is one of the question which is change impact what happens when we change it? Which means there is one design question. There's an edge question there which impact question which means code chunks is a hard coded vector. But ENV embedding different way.

If the M6 inventories built against default without reconciling this what breaks and where. See, there is something wrong in my env file where embedding dimension is something there. But here the embedding dimension is something here the different. So if the embedding dimension does not matches, the PG vector will store their dimension as a part of the column. For example, embedding vector of 51536 dimensions.

Right, but you. But the actual thing is over 2256 dimension. The mismatch will happen because the expected was 1536 but because the env file said 256 the mismatch will happen. It will not pad. It will not trunk to.

It will not differ. Amazing question. It will impact my future stuff. See, it identified a crucial issue and that's where it is asking and making sure that you know about it. So it's telling.

Yes, insert time is going to mismatch. Let's change it. Any one of it has to be changed. So it's a configuration inconsistency. Your schema says vector.

You know, you. You have a few 1536 which I've written over here. But your environment embedding has a 256256 dimension. So that both. Both of the things will not match which is env.

I will not open. But yeah, in ENV have embody embedding model embedding dimension 256 which your model will be using to embed your code chunks. But here it is just 1536 so it will not match. So you should keep in mind. All right, so let's.

Let's. Let's. Let's do one thing thing. Let's come back and I want you to complete everything. Now there's a.

I mean of course in the front. In the. In the starting of the video. I must have already told you that we are accepting challenges. Whoever completes and makes the best AIPR review agent and extends this particular repo and makes a very nice PR will receive as.

As per our prices distribution. So whoever completes it, feel free to tag me up. Feel free to make a pr. Give me the submissions which is up there in the Google form. Now for the people who want me to see completing all the MS.5 which should not be the ideal case.

Guys, you should be able to Quickly complete every single thing by your own. Have more chat with it. Understand every single decision it took. Get your event spine up and running. Get your orchestrator up and running.

Go to up stash Create a free account and get the redis URL up and running. Special express aggregator rag retrieval every single thing is quite easy. You don't need to do anything. You just need to call it out. Go to Codex.

Let's. Let's assume I go to Codex. Okay and what I'm gonna do. Yeah, it's not right now. Not right now is I'm going to click on copy path.

Anytime. I don't care what coding agent you're using. I don't care what you're using. Terminals, that's it. And fix your enbs and you're good to go.

Create your tiger data account right I have listed down the PDF. Feel free to use the PDFs to install everything which is needed. Anytime your team starts Anytime anyone starts the state lives in their thing. Great Append G0 Mark M3 as Pickers Record Event is already exist but there is no backend observability and webhookinggress. So it is building M3 taking something from see it is not trying to implement something new.

It will take something which is already already being there implement constraints it will follow how it will implement the loops it will run alright so I hope it gives a very good idea. Feel free to go ahead. You will get this repository up there and feel free to use that and I hope you will utilize this very sensibly. You will see it's not a wipe coding. You're truly lining understanding every single thing Spent a lot of time into designing this system.

I've spent a lot of time in explaining you the system. Okay so I will see you the next video and I'm trying to get my videos right I'm trying to get my more of the videos. See right now we did not focus much on designing every single component. My next set of videos will focus on designing every single component coding Genesis kit and by the way if you have designed already your Genesis kit will be answer will be very easy. If you do have not designed already your genesis will spoil you up.

So I'll catch up you in the next video. Till then I will see you around.
