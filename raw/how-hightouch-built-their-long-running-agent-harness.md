---
url: https://www.amplifypartners.com/blog-posts/how-hightouch-built-their-long-running-agent-harness
date_fetched: 2026-07-05
backfilled: true
---

# How Hightouch built their long-running agent harness

Amplify is an investor in Hightouch, but this isn’t a promotional blog post, it’s just about some great engineering.

A lot has been said about the future of AI agents and their impact on our economy. Less has been said about how to actually build them.

A few months ago Hightouch released their Hightouch Agents product. It is essentially a **general purpose marketing agent** that can plan campaigns, ask any question or analysis of your data, analyze creative and copy, and automate marketing reporting. 

Though to developers, marketing is often viewed as, well, you know… I can tell you as both a developer and marketer that this is an *unbelievably diverse* set of complex, multi-step, long running tasks that even researchers at frontier labs would shudder at trying to automate. 

The crazier thing is that Hightouch Agents *actually work*. The agent has complete context (e.g. the full customer data mode) thanks to the core Hightouch product, which helps customers connect and take action on all of their marketing data sources like Facebook Ads, Hubspot, etc. And it’s also pre-built with domain expertise on marketing and can reason about complex concepts like creative fatigue, attribution modeling, and incrementality (and maybe getting to the front page of Hacker News). All in all, it’s one of the most advanced agent systems in production today. 

To build it, Hightouch’s engineering team needed to solve a laundry list of interesting context, workflow, and prompt engineering problems that there is no set of commonly accepted solutions for. Based on extensive interviews with their technical team, this post will go through the major components of their agent harness, in particular the idea of **agentic delegation**:

- Separating model planning and execution, and dynamic plan updates
- Techniques for managing context and long-running tasks
- Buffering context to files that can be referenced in the future
- Dynamic subagents that create an isolated execution environment
- Fanning out to smaller, less expensive models instead of embeddings

Let’s get into it.

**Challenges with building long running agents**

So you want to build an agent. Where do you start?

At the time of writing there are a few interesting (if young) agent frameworks to choose from. But when Hightouch started building their Agents product there weren’t, and the common wisdom (if there was any) for building agents was immature. The most common abstraction was borrowed from data platforms: the **Directed Acyclic Graph**, or DAG. The DAG chains together a series of steps like prompting a model, chaining the response with some code, fanning out to more LLM calls, etc. It is a deliberately rigid, deterministic way of thinking.

This works decently for well defined agent contexts with finite decision trees, like handling customer returns. It is a very poor fit, though, for the kinds of open ended tasks that marketers need help from AI with. Hightouch Agents is built to handle prompts like:

- Design a strategy to sell my leftover summer inventory.
- In the last 30 days, what percentage of my new accounts have opened upgraded from free to paid driven by an email?
- How are my new batch of Facebook ads performing?
- Assemble a high-value audience that’s most likely to convert with a 20% sale on our premium plan.

Each of these requires multiple steps, including data retrieval from specific sources, and is highly non-deterministic on a customer-by-customer basis. You can think of these kinds of prompts as **sparse**, meaning that a query from a customer who sells travel software would look nothing like a query from an e-commerce company, and so on and so forth. This agent needs to be a data scientist, a marketer, and a creative partner all in one. 

A naive approach to this problem (and indeed, this was Hightouch’s v0 implementation) would just stitch together prompts and tool calls to the data sources and APIs that a customer has already integrated. You could add a series of system prompts (“You are the world’s best and most handsome data scientist”) and you’d have a decent start. This worked for simple short-lived questions, but would get stuck on more strategic prompts that required complex reasoning.

There are two problems here that speak to fundamental limitations of LLMs. First, naively long LLM calls will **run out of context pretty quickly**. It’s true that context windows are getting longer and longer, but for the purposes of really long running agents current windows are still woefully too small. And second, thanks to instructional fine tuning, today’s models are by **default concise**. They are fine tuned on datasets of short chat sessions, so their default behavior is a poor match for what data science and marketing work really is: open ended, long-context exploration. For many of the kinds of prompts Hightouch would test on, the models would kind of “give up” once they reached a satisfactory answer, in essence stuck at a local optimum. 

This is also why existing agent frameworks weren’t good fits. Most of them were too rigid, focusing more on the developer experience of chaining calls together than on solving the core problem of long-form, autonomous reasoning. The Hightouch team did not need a better way to build a deterministic flowchart with nice LLM integrations. They needed a way to get the model to **think better**.

**Planning, then doing (and re-planning) (and re-doing)**

One of the first major insights the Hightouch team had was explicitly separating the model’s planning from its execution. This is widely understood as a best practice in building agents today, but it wasn’t always this way. Essentially, before you ask the model to do something, you ask it to plan how it would do that thing. For several architectural reasons this tends to lead to significantly better outputs.

Here is how Hightouch implements it:

- A user submits a prompt like: "What are the leading indicators of churn in our customer base?"

- Hightouch takes this prompt and instructs the model to generate a step-by-step plan to answer the question. The model might return a plan like this: - *Step 1: Identify all customers who have churned in the last 6 months.*
- *Step 2: For each churned customer, rewind their history to a point in time just before they churned.*
- *Step 3: Engineer features based on their behavior at that time (e.g., average purchase frequency, product categories viewed, support tickets submitted).*
- *Step 4: Do the same for a control group of active customers.*
- *Step 5: Train a simple classification model to identify which features are most predictive of churn.*
- *Step 6: Summarize the findings and present the key leading indicators.*
 

- This plan is then fed back into the same conversation as the guide for the execution phase. The agent then begins to work through the plan, one step at a time.

OK, simple enough. Where things get really interesting is that Hightouch’s implementation allows the model’s plan to be **dynamic**. As it executes the steps and learns new things grounded in the customer’s data, the model can change its mind and generate a new plan. Not quite continual learning, but it’s a start!

They (cleverly) implemented this via a special set of "system tool calls." Most people think of tool calls as a way for agents to interact with external systems, like running a query or calling an API. But the Hightouch agent harness uses these tool calls to let the agent manage *its own internal thought process* via options like make_plan, execute_step_in_plan, and perhaps most importantly, update_plan. The update_plan tool call might get triggered when the agent discovers a new, unexpected data point, e.g. a large number of churned users recently spent 10+ minutes on a particular screen in the product. At that point the model can decide to break from the original plan and insert a new step to investigate this unusual correlation.

This ability creates a sort of reasoning loop where the agent is constantly assessing its progress against its plan and iterating. In practice there’s a bunch of scaffolding the system needs to make this work beyond just these tool calls. For example, to ensure the plan stays top of mind for the LLM, the harness frequently has the agent "regurgitate" the current plan at the end of its context, since models tend to pay more attention to the most recent tokens in their context window.

**Agentic delegation 1: file buffering and dynamic subagents**

Ah, the context window. This pesky little interface seems to grow tremendously every model generation (we are getting into the millions), and yet it is still painfully too small for all of our grand agent ambitions.

With long running agents the context window is always going to be the fundamental bottleneck. No matter how many tokens a model claims to support, persistent memory remains the biggest unsolved problem in agent building. And for a system like Hightouch Agents, which can involve dozens of exploratory steps and long, complex SQL queries, even a 1M token window can feel claustrophobic.

To solve this, the team developed a series of interesting techniques to effectively manage and compress context. You can think of these little hacks as, in the aggregate, starting to resemble some sort of short and long term memory for the model.

- **Buffering context to files: the agent's scratchpad**

There comes a time in every engineer’s career where they inevitably decide to build a filesystem. Thankfully Hightouch’s engineering team only *sort* of built a filesystem, via giving the agent the ability to read and write context to a temporary, session-specific file system. Here is how it works.

When a tool call or executing a step in the plan returns a large amount of data – this will happen often when the agent executes a SQL query or pulls data from an API – the agent can make a decision. Instead of stuffing the entire result set into context it can call write_file to buffer the response to disk. It keeps a pointer in context with the file name and a brief description of its contents, e.g. churned_users_q2.csv.

Later in the session, the agent can reuse that data with the read_file tool call to pull it back into context. A theme you’ll see a lot in this post is that instead of building some complicated logic tree to decide when the model should do this kind of buffering, Hightouch’s engineering team found that **entrusting the logic to the model itself** led to the best results. It’s guided by some basic heuristics in the system prompt (e.g. “buffer results to a file when the results are large”) but the rest is up to the timeless art of next token prediction. 

- **Dynamic subagents**

Buffering to files works especially well when an agent’s execution step results in a bunch of raw data. But there is another major source of context bloat: the agent’s own reasoning process, which as we’ve already seen Hightouch is trying to *encourage* to be exploratory and long. To handle this the team devised a more elegant solution they call **dynamic subagents**.

The idea is pretty simple: hand off occasional agent tasks and their context to another model. The main agent thread identifies a complex sub-task that will require multiple steps to solve (e.g. the entire feature engineering process from the churn example) and then offloads it:

- Instead of performing said task in the main context, the agent uses a tool call to spawn a separate, isolated LLM thread.
- This isolated thread is given the relevant context and a single, dedicated objective. It then performs all the messy work (writing SQL, exploring data cuts, etc.) within its own self-contained environment.
- Once it arrives at a conclusion, it generates a concise summary and a list of key findings.
- Only this summary is appended back to the main agent's log. All the intermediate "scratch paper" work from the isolated thread is thrown away.

The analogy here is sort of like taking a math test. Your professor wants you to show your work, but…not *all* of your work. Your final answer sheet should be clean and show the most important steps. But to get there, you might use a separate piece of scratch paper for all the messy derivations, calculations, and mistakes. Dynamic subagents is that piece of scratch paper. 

This hierarchical approach allows the agent to perform deep, exploratory analysis on sub-problems without derailing the direction of the main task. Dynamic subagents are worth contrasting with prevailing understandings of micro and macro compaction, because they are able to fully maintain context density without losing any fidelity. Most approaches to compaction patch up the symptoms without addressing the root cause: you’re cleaning up after the fact, as the context fills up with tactical minutiae and *then* you have to go back and decide what to strip out. The Hightouch approach, on the other hand, means the agent can go way further and maintain a high quality context *along the way*. 

**Agentic delegation 2: fanning out, and why embeddings are overrated**

At their core, file buffering and dynamic subagents are about the idea of an orchestrator model delegating out to sub-LLM calls dynamically to manage context effectively. Another way the Hightouch does this is through fanning out. Part of the Hightouch agent harness is its frequent use of fanouts to smaller models instead of relying on a “traditional” (can one even say that?) setup of embeddings in a vector database.

Unstructured data is a central part of Hightouch’s platform and as such a central part of the common use cases customers have for Hightouch Agents. A typical prompt might look something like:

Which types of creative in our Instagram campaigns tend to perform best?

This would be a pretty simple “can you pull the data for me” question if all ad creative was neatly organized into groups in a tabular…sorry, I laughed even writing this. For most companies, it’s actually scattered all over the place and totally unlabeled. A given account might have hundreds of active campaigns, each with multiple images and associated ad copy. How is the agent supposed to figure that out?

The textbook answer in 2024 would be embeddings. You create vector embeddings for all your ad creatives, store them in a vector database, and then, at query time, create an embedding of the user's request and perform a similarity search. But the issue with embeddings is that they’re actually quite dumb. They lack the intelligence that modern multimodal LLMs have, which are able to *reason* about their prompt and what’s in the image in a much more intelligent way than naive vector operations. So instead, Hightouch opted for a brute-force approach that is both simpler and, in their context, more effective: a **fan-out pattern** using small, cheap LLMs.

In their agent harness, the main orchestrator agent acts as a dispatcher. It uses a tool that essentially says, "for every campaign in our database, run the following analysis." It then spawns hundreds of parallel API calls to a much smaller, faster, and cheaper model, like Anthropic's Haiku. Each of these micro-LLM calls is given the creative assets for a single campaign and a very specific, dynamically generated question: "Does the attached image contain user-generated content? Answer yes or no." Or, "Analyze the color palette of this image. Is it generally bright or dark?"

Things have come a long way over the past few years, and these small models are surprisingly good at these targeted classification tasks. Their responses are simple "yes" or "no" answers in structured JSON and are then collected and aggregated. And voila, in under a minute, the main agent has a perfectly structured dataset mapping every single campaign to the attributes the user asked for, which it can then use for the next step of its analysis.

All in all this process is way cheaper, easier, and more reliable (for this kind of task) than going through the rigor of setting up a proper RAG system.

First there was prompt engineering, then there was context engineering. The one thing both of these nonsensical phrases share is that they have nothing to do with actual engineering. *Real* context engineering is exactly what I wrote about here: buffering to files, planning and execution loops, tool calls, and dynamic subagents. 

The story of Hightouch Agents is essentially one in the realities of applied AI engineering. The ugly truth is that creating a truly capable agent system today has less to do with scaling RL or exotic model architectures and more to do with the deeply unfussy work of *actual* context engineering.

And if this kind of work is interesting to you, Hightouch’s engineering team is hiring.
