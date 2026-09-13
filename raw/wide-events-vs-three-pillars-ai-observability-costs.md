---
url: https://www.honeycomb.io/blog/wide-events-vs-three-pillars-ai-observability-costs
date_fetched: 2026-09-13
---

# Wide Events vs. Three Pillars: AI Observability Costs

AI agents make telemetry costs harder to predict. This post compares the three pillars against the wide event model, and explains why wide events keep AI observability costs predictable without sacrificing the context engineers need.

By: Nick Travaglini

#### How Observability Makes Better AI and Human Investigators

A hands-on, on-demand workshop from Honeycomb and AWS on how the same telemetry that helps engineers investigate distributed systems gives AI agents the context to reach application-level root cause — and how to make the agents you build observable in production.

Watch NowAs agentic AI workflows gain traction within organizations, those organizations are asking how to account for their behavior while keeping costs manageable. Some are sticking with the old three pillars of observability approach: take a measurement to create a metric, record output to a log, and track serial progress with a trace. Each of these is useful, but treating them as distinct formats from the start means paying for them distinctly too. Separate storage doesn't come cheap. Add the extra software that makes up an AI agent along with the added unpredictability of non-deterministic systems and costs can balloon.

As an alternative to the idea of observability pillars, other organizations are using wide events. Wide events are a single format that can handle all three types of data because they're all one format. These organizations are consolidating their views to a "single pane of data," as we like to say, and are saving money without sacrificing the context they need to account for their agents.

In this post, I'll build upon a prior paean to wide events. First, I'll describe the source of ballooning costs and why it's a result of centering distinct pillars of observability. Then, I'll explain how wide events prove more useful and cost effective in general, and for AI agents specifically.

# Read our O’Reilly book, Observability Engineering

Get your free copy and learn the foundations of observability,

right from the experts.

## AI is just software

Here's the honest truth: AI is software. Unpredictable, but software nonetheless. That means engineers can use telemetry data to identify problems that lead to poor customer experiences. Given that unpredictability, engineers need lots of context now more than ever if they hope to stand a chance of finding those problems. That requirement need not be a problem on its own. But if these systems need three types of data to present an adequate picture, which is analogous to needing three distinct languages to get an adequate description of something, then each type of data is going to see its costs grow. That's not even considering the headache of the additional overhead of trying to reconcile them, which isn't possible because the distinction is required by hypothesis.

What is this additional context, and why might each type of data see its cost grow? AI agents add a whole new set of components to keep track of besides the dimensions of software people may be used to like user ID, http response code, enabled feature flags, etc. An agent and its non-deterministic behavior may be due to which model was used, which skills were invoked, whether a particular tool call worked and what failovers the model 'thought' to try, the initiating prompt, and more.

Now, imagine tracking the number of each operation in distinct time series (including lots of one-offs for singular inputs like prompts) in a metrics system, a readout of each component every time it runs in their own log files, plus a trace for every turn in a conversation. That's a lot of data, only some of which conceptually overlaps like the recording of a unique prompt as the lone datapoint in a dedicated time series, the actual prompt text in a log, and the initiation point of an agent's trace. And then there's the additional work to actually correlate them all...

The first instinct in a situation like this may be to cut things like retention time. If data isn't stored for as long, creating a bunch of data may not be so bad. This assumes that storing the data is a significant cost, which it typically isn't. Instead, ingest and the compute necessary to run queries (in an acceptably fast manner) tend to be the dominant drivers of cost. Since AI agents are non-deterministic and therefore outputs are statistical, organizations are going to want long-term data so they can suss out behavioral patterns that only emerge with large numbers. It may seem like the only solution is to suffer through limited context... Fortunately, it doesn't have to be that way!

## Enter wide events

The wide event model makes it possible to have all of that with just one data file. Let's review what a wide event is. In her blog post on observability 2.0, Charity describes wide events in the following way:

- *Data gets stored in arbitrarily-wide structured log events (often called "canonical logs," or what AWS internally refers to as "service logs"), often with trace and span IDs appended.*
- *You can visualize the events over time as a trace, slice and dice your data to zoom in to individual events, or zoom out to a birds-eye view.*
- *You can interact with your data by GROUP BY, break down, etc.*

*Aggregation is done at read time and preserves raw events for ad hoc querying. Hopefully, you derive your SLO data from the same data you query! Think of it as BI for systems/app/business data, all in one place. You can derive metrics, or logs, or traces, but it's all the same data.*

Breaking this down, an event is a file where the information stored within it is structured in some standardized pattern (Honeycomb uses a flat structure, no nesting allowed, and patterns the data as Key:Value pairs). That file is called an "event" because the instrumentation's operation of creating the information to go into the file is itself an event. In fact, it's the event that makes the information possible. The "wide" part comes from counting the number of aspects or properties of the system recorded in the file. If it's a big number, it's wide, and if it's a smaller number, it's narrow. The point, though, is that the only limit to how wide the event can be is the practical limit of the system ingesting the file.

If this sounds like a description of a log file, you're correct! This format is damn useful because of its versatility. Charity listed several different ways to use the files and the information they record, including as metrics, logs, and traces. Since engineers can use the wide event model in so many ways, that also means that the organizations they work for only pay to store the data they need once. When organizations stop making arbitrary distinctions and assuming conflicts, like logs vs metrics, they can get down to the business of saving money and engineering effort.

## Savings costs where agents are concerned

Let's say I run an e-commerce website and have a contextual chat system. That system knows what product page my prospective customer is looking at, and it includes a skill which has the agent synthesize submitted reviews of that product. The chat can act as a proxy, and my prospective customer can ask that proxy about the product as if they were asking someone they knew who'd already bought it.

In the three pillars model, this would require several metrics time series recording information like the number of turns in the conversation, the number of times the skill was invoked, the length of time it took to synthesize the reviews, how long each response took, the p95 of all response times on this page over the last week, how many other tool calls were necessary, etc. The output of any and probably most (if not all) operations will get their own log file. Finally, each conversation may be its own trace but that could be quite unwieldy; it might be wise to split it up so each turn in the conversation gets its own trace.

Now, compare the wide event model. Each operation still generates a file, but because they're wide events, those files can include information that allows recording metrics directly like the operations' duration and for calculating aggregation like p95 on the fly. With something like the "parent" concept, they can also store information for composing them into traces. And when an agentic conversation featuring an agent and sub-agents requires a view of multiple traces, they can even build up into an Agent Timeline. All of that power comes from a single data format that organizations only need to pay to store once.

## Versatility and simplicity

The versatility of wide events comes from their simplicity. It's that same simplicity that makes them not only more cost effective than the pillar model, but also more predictable. Undoubtedly, the three pillars model, especially when metrics are first among equals, can be cheaper in some cases due to preaggregation. However, it's the inherent uncertainty of whether your case is one of those "some cases" that's the problem.

If you need lots of context, like with AI agents, then it may not be feasible to preaggregate enough data to make the cost-savings worthwhile. Wide events don't require you to trade off between context and preaggregation, and that's why their simplicity enables cost predictability.

If you want to make your AI observability costs more predictable, start with examining what each of your pillars of observability are collecting and where there's duplication between datastores—and if it's a lot, start looking into shifting to the wide event model. As you make that shift, here's what you'll want to be thinking about:

- Identify which information must remain available at request level and which can safely be aggregated or derived.
- Consider how cardinality, retention, workload growth, etc. affect the ability to forecast spend.
- Where useful, connect token and model activity to telemetry volume so teams can understand how AI workload growth affects observability spend.

The point is to give yourself a means for predictable growth while preserving the information your team needs to make sure your AI agents are accountable to you.

Finally, consider using Honeycomb as your observability tool of choice for those wide events. Honeycomb was designed from the beginning to use wide events and helps many customers understand their (AI) application behavior without forcing a tradeoff between their required context and cost.
