---
url: https://www.oreilly.com/radar/agent-memory/
date_fetched: 2026-07-05
backfilled: true
---

The following article originally appeared on Angie Jones’s LinkedIn page and is being republished here with the author’s permission.

I’m fascinated by the concept of agent memory. LLMs are stateless by design, meaning they have no memory or awareness of past interactions. Each prompt you send to an LLM is treated as a completely isolated event.

When you have a continuous chat with an AI agent, it feels like the AI remembers previous messages. However, the interface itself is faking it. Behind the scenes, your agent takes the entire conversation history and resends *all of it* to the LLM as one giant, combined prompt.

Companies, researchers, and even indie devs are all trying to crack agent memory. Because once an agent can remember, the entire interaction changes. It can build on what it learned, adapt to the user, resume work after a restart, and develop a sense of continuity.

Recently, I spent time with Richmond Alake, who has been in the trenches working on agent memory at Oracle.

We talked about the different kinds of memory, why memory is harder than it sounds, and what it takes to build a memory system that is actually useful in production.

That conversation made something very clear to me. When people say, “agent memory,” they often mean very different things.

So let’s unpack the various types of memory.

## Conversational memory

Conversational memory is the one most people think of first. It stores the messages exchanged between the user and the assistant.

This makes sense. If I ask, “What did I say was the ultimate goal of this task?” the agent needs access to the conversation in order to answer. Without that history, every turn starts from zero.

But this is also where many memory systems go wrong.

The most common first attempt is to keep appending prior messages to the prompt. For example:

User: I’m building a customer support agent.

Assistant: Great, what should it do?

User: It should look up past tickets and draft replies.

Assistant: Got it.

User: Also, I prefer Python and FastAPI.


Then on the next call, we send all of that back to the model along with the new question.

This works for a short conversation, but the agent only “remembers” because we keep reminding it. This is not really memory engineering.

Eventually, the conversation gets too long and the model receives a giant blob of context where some details are important, some are stale, and some are completely irrelevant. The agent may technically have the information, but that doesn’t mean it can use it well.

So yes, conversation history is a valid and important type of memory. But it shouldn’t be the whole memory strategy. Real agent memory requires deciding what should be stored, where it should be stored, how it should be retrieved, and when it should be summarized, forgotten, or compressed.

## Semantic memory

Semantic memory stores durable facts.

These are things that should outlive the exact conversation where they were learned:

- The user prefers Python over TypeScript for backend work.
- The customer support agent needs access to past tickets.
- The production system handles 50,000 queries per day.

This is different from conversational memory because the exact wording and sequence are less important. What matters is the meaning.

If the agent needs to recall what stack the user is using, it should retrieve the memory even if the user never says those exact words again.

Vector search is useful for this. The memory can be embedded and retrieved by semantic similarity.

The benefit is that the agent doesn’t need to replay the full conversation. It can retrieve the few durable facts that are relevant to the current request.

## Episodic memory

Episodic memory stores events.

This is the “what happened” layer of memory:

- The agent searched the web for recent API gateway patterns.
- The agent generated a draft response for ticket #4821.
- The workflow failed at the compliance review step.

Episodic memory is especially useful for debugging, auditing, and long-running workflows.

For example, if an agent makes a decision, I may want to know what happened right before that decision (e.g., What tools did it call? What data did it retrieve?).

This type of memory often benefits from structured storage.

For example:

Find all failed tool calls from the mortgage approval workflow in the last 24 hours.


That is a database query problem, not just a vector search problem.

## Procedural memory

Procedural memory is about how to do things.

For example:

- When investigating a failed deployment, check logs first, then recent config changes, then dependency updates.
- When drafting a customer support reply, include the ticket summary, likely cause, recommended fix, and next step.
- When creating a database-aware agent, scan table comments, column comments, constraints, and recent workload patterns.

This is the kind of memory that helps an agent improve its process. That’s powerful because agents are often asked to operate in messy real-world environments. With procedural memory, it can reuse proven approaches.

The value extends beyond just knowing things to actually knowing how to proceed.

## Entity memory

Entity memory stores facts about specific people, accounts, projects, systems, tickets, or objects.

For example:

- Angie prefers practical examples over abstract explanations.
- Customer Acme Corp has strict data residency requirements.
- Ticket #4821 is related to a billing reconciliation issue.

Entity memory matters because many agent tasks are scoped around a particular thing.

If I ask, “What do we know about Acme Corp?” I don’t want every memory in the system. I want memories attached to that customer.

This is also where memory safety becomes important.

Agents should not accidentally mix memories between users, customers, or projects. A memory system needs strong scoping so one user’s context does not leak into another user’s response.

## Working memory

Working memory is the short-term scratchpad for the current task.

This is where the agent keeps temporary information while reasoning through a problem.

Working memory is usually not meant to last forever. It’s useful during the task, but it may not deserve to become durable memory.

If an agent stores every temporary thought as long-term memory, the memory store gets noisy very quickly. The agent may later retrieve half-baked assumptions as if they were facts, which is dangerous.

Not everything the agent observes or thinks should be remembered permanently.

## Summary memory

Summary memory is one many agent users are familiar with. It deals with the problem of context windows being limited.

Even with large context models, you can’t keep appending forever. At some point, you need to compress.

Summary memory stores a compact version of a longer thread or context window. The original details can still live in the thread, but the prompt gets a smaller representation.

For example, instead of sending 80 turns of conversation, the agent might send:

The user is building a SaaS customer support agent. They prefer Python and FastAPI, deploy on OCI, and want the agent to retrieve past tickets before drafting replies. They are currently evaluating memory strategies for production usage.


## Why memory is hard for agents

At first, memory sounds straightforward: store things, retrieve them later.

But the hard part is judgment, not storage.

What should be remembered? If the user says, “I usually prefer Python,” that’s probably worth remembering. If they say, “Let’s try Python for this one experiment,” maybe not. The agent needs to distinguish durable details from temporary context.

When should memory be updated? People change their minds, and systems and requirements change. If a user used to prefer FastAPI but now works mostly in Java, should the old memory be deleted, overwritten, or kept with a timestamp? A memory system needs a correction strategy.

How much memory should be retrieved? Retrieving too little means the agent misses important context. Retrieving too much means the prompt becomes noisy. This balance matters as more context isn’t always better.

How do we prevent memory leaks? If memories are shared across users, agents, or tenants, scoping is critical. The agent should only retrieve memories it’s allowed to use. This is especially important in enterprise systems where agents may operate across many customers, teams, or workflows.

How do we know whether memory helped? Memory should improve the agent’s behavior. It should reduce repeated questions, improve continuity, lower token usage, and help the agent produce more relevant responses. If memory just adds complexity without improving outcomes, it isn’t doing its job.

### How Oracle is approaching agent memory

Richmond was gracious enough to share how Oracle is tackling this with the Oracle AI Agent Memory Package (OAMP), built on top of Oracle AI Database 26ai.

Yes, an AI database! Think of it as a database that can store and query the kinds of data AI applications need, not just rows and columns. That includes embeddings and JSON documents along with text search and regular SQL. These live together in the database, so an agent does not have to bounce between separate systems just to gather context.

The idea is to make Oracle AI Database the memory core for agents. Instead of stitching together a vector database, a relational database, a document store, and custom thread management, OAMP provides agent-friendly memory primitives on top of a database that already supports multiple data access patterns.

At a high level, OAMP gives you:

- Users and agents to scope memory ownership
- Memories for durable facts and extracted knowledge
- Threads for conversation history and continuity
- Context cards for compact, prompt-ready memory retrieval
- Summaries for long-running conversations
- Vector search for semantic recall
- Database-backed persistence so memory survives restarts

This matters because, again, agent memory is not only a vector search problem. Some memory needs semantic retrieval. Some need ordered reads or exact SQL filtering. A database-backed memory system gives you room to support all of those patterns.

Here’s a small example of what that looks like in code:

```
from oracleagentmemory.core import OracleAgentMemory
from oracleagentmemory.core.llms import Llm
client = OracleAgentMemory(
    connection=connection,
    embedder="text-embedding-3-small",
    llm=Llm("gpt-5.5"),
    extract_memories=True,
    schema_policy="create_if_necessary",
)
client.add_user(
    "angie",
    "Developer exploring agent memory patterns."
)
client.add_agent(
    "memory-demo-agent",
    "Assistant that demonstrates Oracle AI Agent Memory."
)
client.add_memory(
    "Angie is fascinated by agent memory and prefers practical examples over abstract explanations.",
    user_id="angie",
    agent_id="memory-demo-agent",
)
```
There are a few important ideas packed into this snippet.

The OracleAgentMemory client is the bridge between the agent application and Oracle AI Database. The database connection tells OAMP where memory lives. The embedder tells it how to turn memory text into vectors for semantic retrieval. The LLM enables automatic memory extraction and summary generation. And `schema_policy="create_if_necessary"` lets OAMP manage the underlying memory schema instead of making every application reinvent it.

The user and agent registration may look like simple setup code, but it’s actually part of the memory model. Memories need ownership. In a real system, you don’t want one user’s preferences showing up in another user’s session, and you don’t want memories written by one agent casually mixed with another agent’s context. The user ID and agent ID give the memory layer a way to scope what gets stored and retrieved.

The `add_memory()` call stores a durable fact. This is a piece of information the agent may need later, even if the exact conversation has moved on.

Given this, we can now recall memories.

```
results = client.search(
    "how should I explain this topic to Angie?",
    user_id="angie",
    max_results=3,
)
```
This `search()` call shows the part that makes semantic memory useful. The query doesn’t have to match the stored sentence exactly. We stored that I prefer practical examples, but we searched for how to explain something to me. Those are different words but related in meaning. That’s the point.

## Threads and context cards

Durable memories are only part of the picture. Agents also need conversation continuity.

With OAMP, a thread can represent a real work session, such as an agent helping investigate a production issue:

```
from oracleagentmemory.apis.thread import Message
thread = client.create_thread(
    user_id="angie",
    agent_id="support-triage-agent",
)
thread.add_messages([
    Message(
        role="user",
        content="Customer Acme Corp is seeing intermittent checkout failures after the latest deployment.",
    ),
    Message(
        role="assistant",
        content="I'll check recent deployment notes, related incidents, and payment service logs.",
    ),
    Message(
        role="user",
        content="Focus on the payment gateway first. We saw similar timeout errors last quarter.",
    ),
])
```
This is much closer to how memory shows up in real agent applications. The useful context is not just that messages were exchanged. It’s that this thread is about Acme Corp, checkout failures, a recent deployment, the payment gateway, and a related incident from last quarter.

When it’s time to call the model, instead of passing the entire raw thread, you can ask for a context card:

`card = thread.get_context_card()`The context card gives the agent a compact block of relevant memory to use in the next prompt.

Conceptually, the prompt becomes:

System: You are a helpful assistant. Use the provided memory context.

Memory context: [context card]

User: What did we decide earlier?


This is a much cleaner pattern than appending every message forever.

### Automatic memory extraction

OAMP can also extract memories from conversation.

For example, if the user says:

I prefer Python over TypeScript for backend work. I usually deploy FastAPI apps on OCI behind an API gateway.


The memory system can extract durable facts such as:

The user prefers Python over TypeScript for backend work.

The user deploys FastAPI applications on Oracle Cloud Infrastructure behind an API gateway.


That means the application does not have to manually call `add_memory()` for every useful fact.

A smart thread can be configured like this:

```
thread = client.create_thread(
    user_id="angie",
    agent_id="memory-demo-agent",
    memory_extraction_frequency=2,
    memory_extraction_window=4,
    enable_context_summary=True,
    context_summary_update_frequency=2,
)
```
This tells the system to periodically inspect recent messages, extract durable memories, and maintain a running summary.

Here is where agent memory starts to feel more like a living part of the agent architecture vs just a data structure.

## Teaching an agent about a database

One of the most interesting examples Richmond and I discussed was using memory to teach an agent about a database.

Imagine an enterprise data agent that needs to answer questions about a schema it has never seen before. Instead of fine-tuning a model, the agent can scan the database catalog and store what it learns as memory.

It might inspect:

- ALL_TABLES for table names and row counts
- ALL_TAB_COLUMNS for column names and types
- ALL_TAB_COMMENTS for human-written table descriptions
- ALL_COL_COMMENTS for column descriptions
- ALL_CONSTRAINTS for primary keys and foreign keys
- V$SQL for recent workload patterns

Then it can convert those technical details into natural-language memories.

For example:

Table SUPPLYCHAIN.VESSELS stores individual ships owned or operated by carriers. It includes vessel identifiers, carrier relationships, and operational metadata.


Now when a user asks:

Where would I find information about ships and carriers?


The agent can retrieve the relevant schema memory by meaning.

This is a beautiful pattern because it avoids one of the common traps with agents expecting the model to already know your private system.

It doesn’t. And that’s okay.

You can teach it by turning your system’s metadata into memory.

The more I learn about agent memory, the more I believe this will be one of the defining pieces of agent architecture.

Tool calling lets agents act. Planning lets agents decide what to do. Memory lets agents build continuity.

With memory, we can start designing agents that feel less like one-off prompt responders and more like persistent collaborators.

Of course, this also raises the bar. Memory has to be scoped, auditable, correctable, and intentionally retrieved. Bad memory is worse than no memory. So the challenge is not simply giving agents memory but giving them the right memory architecture.

Oracle’s OAMP approach is one way to make that system concrete: users, agents, memories, threads, context cards, summaries, and database-backed retrieval.

And while the implementation details matter, the bigger idea is that if we want agents to be useful beyond a single prompt, they need a way to remember.

Not everything. But enough to carry context forward.
