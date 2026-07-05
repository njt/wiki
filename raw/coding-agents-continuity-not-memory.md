---
url: https://oldskultxo.substack.com/p/maybe-coding-agents-dont-need-a-bigger
date_fetched: 2026-07-05
backfilled: true
---

# Maybe Coding Agents Don't Need a Bigger Memory. Maybe They Need Continuity.

### A practical reflection on why coding agents lose the thread between sessions, and why the repository itself is the right place to preserve it.

I used to think the problem was memory. That was the obvious diagnosis.

Every new coding-agent session started with the same ritual. Open the repository. Read the README. Inspect the project structure. Search for the files that looked important. Reconstruct the task. Guess which commands mattered. Ask again what had already been tried. Then do the actual work.

Sometimes.

Because a lot of the work was not work. It was orientation.

A coding agent can have a large context window and still lose the operational thread. It can have chat history and still fail to know what happened in the last run. It can retrieve semantically similar notes from a vector store and still miss the one fact that mattered:

- this command already failed. 
- the previous session stopped here. 
- this file looked relevant, but it was a dead end. 
- the validation was not actually run. 

One day I stopped thinking about the problem as “agent memory”. That word was too broad. Too attractive. Too dangerous.

Because once you say memory, the temptation is to build a bigger one.

A bigger context window. A bigger note store. A bigger vector database. A bigger archive of everything the agent has ever seen, said, touched, generated, or vaguely implied.

That sounds powerful, but it is also how you build a very expensive junk drawer.

**Context is not continuity**

Context is what the agent has available now.

Continuity is what lets the next execution continue from what actually happened before.

Those are not the same thing.

Long context helps while a session is alive. It gives the model more text to work with. More files. More prior messages. More implementation details. More room. Although it is really useful it does not automatically produce continuity.

When the session ends, gets compacted, moves to another tool, switches from one coding agent to another, or simply starts tomorrow from a fresh prompt, the problem comes back, and the same questions are there:

- What was active? 
- What changed? 
- What failed? 
- What was validated? 
- What was only assumed? 
- What did the previous run leave unresolved? 

A larger context window can hold more text, but it does not turn execution history into structured project state.

The problem is not only loading enough information. The problem is preserving the right kind of information.

**The wrong abstraction: “remember everything”**

The straightforward version of agent memory looks something like this:

```
repository
↓
chat history
↓
large prompt
↓
agent execution
↓
summary
↓
next large prompt
```
It works for a while. Then it starts to rot.

Summaries become too broad and notes become stale. Old assumptions remain mixed with verified facts while failed approaches sit next to successful ones with the same visual weight.

The agent retrieves something that sounds related, but nobody knows whether it is current, useful, contradicted, or just a nice-looking hallucination from three sessions ago.

This is where “more memory” can be worse than no memory because the agent needs operationally trustworthy information and not only related information.

The general idea may be the same, but the difference between a memory item and a continuity record is huge if we talk about usefulness. One is a recollection. The other is a handoff.

A memory item like this is weak:

`We probably fixed the parser by changing the tokenizer.`A continuity record like this is much stronger:

```
Task: fix parser edge case
Files edited: src/parser/tokenizer.py, tests/test_parser.py
Command run: pytest tests/test_parser.py
Result: passed
Known failure: full test suite still not executed
Next action: run full parser test group before extending scope
Evidence quality: partial
```
**Instructions, vector memory and chat history are not enough**

**Repository instruction files** are useful because they can tell a coding agent:

```
how to run tests
which style to follow
where the main modules live
what conventions matter
what not to touch
```
That is good project context. But instructions are mostly static.

They explain how to work in the repository. They do not usually know what happened ten minutes ago.

They do not know that a task is paused or that the last validation failed. They do not know that a previous attempt edited the wrong layer. They are not aware of a user interrupted the session and narrowed the scope.

A coding agent needs both: stable instructions and changing work state.

One without the other is incomplete. They have different job.

*Instructions make the agent behave better in the repo and Continuity makes the agent continue better in the repo.*

**Vector databases** are good at retrieving semantically similar chunks. That is valuable when the problem is:

```
Find related documentation.
Find similar past notes.
Find chunks of a large knowledge base that match this question.
```
But coding-agent continuity is not only a semantic retrieval problem.

The most important continuation facts are often small, boring, and operational:

```
npm test failed with TS2322
pytest passed only for tests/test_parser.py
migration file was inspected but not edited
user said not to touch auth middleware
branch was dirty
validation was skipped
```
A vector store may retrieve something related. But relation is not enough.

The next session needs provenance.

- Was this observed? 
- Was it inferred? 
- Was it claimed by the agent? 
- Was it validated? 
- Was it contradicted later? 
- Is it still fresh? 

That is where generic memory becomes weak. Not because semantic retrieval is bad, but because execution continuity needs lifecycle, timestamps, quality signals, and explicit evidence boundaries.

*A vector database can be part of a larger system but it should not be mistaken for the system.*

**Chat history** is another piece of the map. It is also not enough, even it feels like continuity because it contains the conversation.

But a conversation is not an execution state.

It includes the useful facts, yes. But it also includes:

```
abandoned ideas
intermediate reasoning
outdated plans
user corrections
speculative explanations
old constraints
generated code that may or may not have been applied
"we should do X" statements that never became real
```
The next agent can read all of that and still not know what is true now.

Or even worse, chat history is often bound to one provider, one tool, one account, one session, or one UI.

But repositories outlive chats.

The state that matters should live with the project.

*Continuity should not belong only to the conversation. It should belong to the repository.*

**The repo is the natural boundary**

For coding agents, the repository is already the operational unit.

It contains the code, the tests, the build system, the conventions, the current branch, the diff, the file structure, the commands, the failures, and the artifacts of work…

So the repository seems to be the correct place for continuity to live too.

Continuity should not become a hidden cloud memory or a provider-specific black box.

Continuity should live as local and inspectable artifacts.

That has several advantages.

- The user can review them. 
- Another compatible agent can read them. 
- The continuity layer can be cleaned, corrected, compacted, or ignored. 
- The memory is no longer trapped inside one chat session. 
- The system can distinguish between local-only operational state and a safer portable subset that could be shared through Git (this matters for teams). 

**What coding agents actually need between sessions**

After working through several iterations of a repo-local continuity runtime on a large production codebase, the useful pieces became much clearer.

A new session does not need a memoir, it needs a compact execution surface.

Something like:

```
resume
-> active work state
-> relevant decisions
-> known failures
-> validation expectations
-> structural entry points
-> stale / unverified warnings
-> next action
```
Then the agent works.

And finally the agent records what matters about the session:

```
work
-> files touched
-> commands executed
-> tests observed
-> failures learned or resolved
-> decisions made
-> unresolved risks
-> next handoff
```
Then the next session resumes from that.

Not from vibes. Not from “I remember we were doing something with auth”. Not from a 40-message chat transcript.

From explicit operational continuity, the loop looks simple:

`resume -> work -> finalize -> resume`That loop turned out to be more important than any individual memory feature.

Because without a lifecycle, memory is passive. It waits. It accumulates… It becomes archaeology.

With a lifecycle, memory becomes part of execution.

The agent starts from bounded state, does work, then records factual evidence for the next run.

That changes the shape of the problem.

**The useful memory is small**

This was counterintuitive.

The better the continuity layer became, the less I wanted it to return.

At the beginning, the instinct was to load more of everything: decisions, handoffs, files, notes… Until the context started to feel heavy again. Different packaging. Same problem.

So the goal changed.

The continuity layer should minimize useless rediscovery.

That means the best resume payload is not the largest one. It is the one that gives the agent enough operational grounding to avoid starting cold.


A good resume should answer:

```
What is active?
Where should I start?
What should I avoid repeating?
What validation matters?
What is stale or unproven?
What should I do next?
```
That is it. The rest can be available on demand.

The agent does not need the entire memory palace before touching one file, it needs the right door.

**Continuity needs provenance**

This of the strongest lessons to learn is that Continuity quality matters more than memory volume.

The source of truth should not rely on memory, stale handoffs, or the previous agent summary.

If a continuity layer cannot say “this is stale”, “this is unverified”, “this was demoted”, or “this lacks validation evidence”, then it is too trusting and that is dangerous.

The stronger model is evidence-weighted continuity:

```
runtime observed this
agent claimed this
validation supports this
user corrected this
later work contradicted this
this part is still unknown
```
That framing changes the agent’s relationship with memory because it changes the goal, from believe memory to use it with the right weight.

A good continuity layer should make uncertainty visible, because hidden uncertainty is where bad continuations begin.

**What changed when I tried to build it**

The tool did not begin as a full continuity runtime. It began with a simpler idea:

*Move reusable context out of the prompt.*

Then each layer exposed the weakness of the previous one. The rabbit hole was not adding features; it was discovering which kind of state actually mattered:

- External memory solved repetition, but introduced noise. 
- Structured context reduced noise, but still lacked execution state. 
- Failure memory captured pain, but not active work. 
- Work State captured active work, but needed validation semantics. 
- Execution contracts added guidance, but needed quality and compliance signals. 
- MCP improved access, but required safety boundaries. 
- Guardrails helped at action boundaries, but had to stay compact. 

The direction became clearer over time:

```
less generic memory
more operational continuity
less hidden state
more inspectable evidence
less "remember everything"
more "resume from what matters"
```
That is the core architectural lesson.

**Failure memory was the first real turning point**

Remembering failure is often more useful than rememberin successful work.

A coding agent wastes a lot of time by repeating plausible mistakes.

- It opens the wrong file because the name looks right. 
- It runs the same command that failed before. 
- It tries the same fix because the error message resembles something common. 
- It follows a path that was abandoned two sessions ago. 

A failure memory layer changes that.

Not by saying “never do this again”, but by saying:

```
This failed before.
Here is the command.
Here is the error.
Here is the area.
Here is whether it was later resolved.
Treat this as context, not truth.
```
That last sentence is important because failure memory should not make the agent afraid of the repo. It should make it less naive instead.

There is a difference.

**Work State changed the model**

While failure memory handles recurring friction, Work State handles unfinished work.

That distinction is essential.

A handoff summary is useful, but it is not the same as an active task.

A decision record is useful, but it is not the same as suspended execution.

A failure pattern is useful, but it is not the same as “we are currently halfway through this change and validation is still pending”.

Work State should preserve the live thread:

```
task
hypothesis
relevant files
current status
next action
risks
recommended validation
unverified gaps
branch context
```
This is where continuity starts to feel operational rather than documentary.

**Execution contracts are not bureaucracy**

Continuity needs to do more than describe the past. It needs to shape the next execution.

Not control it. Not sandbox it. Not block normal work.

Just guide it.

I add execution contracts to the tool so that Continuity could guide the next session.

A resume can include a compact contract:

```
first action: inspect tests/test_parser.py
edit scope: parser and parser tests
validation: pytest tests/test_parser.py
finalize: record observed commands and unresolved gaps
strength: soft
```
This is not a law. It is a route, and if the route is not followed, that becomes a signal. When working with agents, it is really difficoult to lead with certanity. The agent can simply not follow a clear route because it had a good reason, or because the contract was too strict.

Anyway, what matters is that the system can compare expected execution with observed execution.

Not perfectly. Not magically. But enough to surface gaps:

```
canonical validation was not observed
edited outside expected scope
first action was skipped
finalize was missing
```
That is much more useful than pretending the session went fine because the final answer sounded confident.

**MCP made the boundary cleaner**

A CLI lifecycle was enough to prove the idea but modern coding-agent workflows increasingly need tool-level integration.

This is where exposing continuity through local MCP tools and resources makes sense, because it gives compatible agents a cleaner interface:

```
resume continuity
prepare focused task context
inspect lifecycle status
check continuity quality
run guard before risky boundaries
finalize observed work
```
The important design point is that this does not require a cloud service.

It does not need arbitrary shell access.

It does not need a hidden daemon.

It can remain local-first.

The CLI remains the source of truth, while MCP becomes a more ergonomic bridge for agents that can use it.

That is important because continuity should be available to the tools developers already use not locked behind one harness.

**Guardrails should be compact**

Another trap: once continuity exists, it is tempting to make the agent check everything all the time.

```
Before every edit.
Before every read.
Before every command.
Before every thought.
```
That would be overkilling, and expensive.

Guardrails should appear at boundaries:

```
before first edit
before risky command
before final answer
before finalize
when scope changes
when switching agents
when continuing after idle or restart
```
A guard should not return the whole memory again.

It should answer a compact question:

Is this action aligned with current continuity?

Possible outputs can be simple:

- allow 
- caution 
- re-ground 
- block 

Most of the time, the answer should be allow, sometimes caution and occasionally re-ground. But rarely block.

If the guardrail becomes louder than the work, the system has failed.

**The economics are not free**

Continuity has overhead.

- A resume payload costs tokens. 
- A finalize summary costs tokens. 
- A guard call costs attention. 
- A local runtime has complexity. 

For a one-shot task, it may not be worth it.

That is fine. Not every task needs continuity.

The value curve usually starts to make sense when:

- The task spans several prompts 
- The work crosses sessions 
- The repository is large enough to make rediscovery expensive 
- Failed commands matter 
- Validation paths matter 
- More than one agent or tool may be involved 
- The user is tired of repeating the same project explanation 

The pattern I have observed is quite simple:

```
1–2 prompts probably not worth it
3–7 prompts break-even zone
7+ prompts increasingly useful
multi-session strong use case
cross-agent very strong use case
```
This is not a universal benchmark. It depends on repository size, task type, agent behavior, and how disciplined the lifecycle is.

Continuity pays for itself when it prevents repeated orientation, wrong-path exploration, and re-derivation of previous decisions.

That is the real saving.

Not “fewer tokens” in isolation.

Less wasted cognition, repeated setup and operational amnesia.

**Bigger memory is still useful**

This is not an argument against long context, vector databases, instruction files, chat history or agent-specific memory.


All of them can be useful. But they solve different parts of the problem.

- Long context helps the current session reason over more material. 
- Instruction files encode stable repository behavior. 
- Vector search retrieves related knowledge. 
- Chat history preserves conversation. 
- Agent-specific memory can improve one tool’s experience. 

Continuity does something else.

It preserves the operational thread across execution boundaries.

That makes it complementary. Not a replacement.

The mistake is treating all of these systems as the same category because they all use the word “memory”.

They are not the same category.

A coding agent does not only need to know more. It needs to continue better.

**The practical architecture**

The architecture I would now choose is boring on purpose:

```
repo-local artifacts
├── active work state
├── handoffs
├── decisions
├── failure memory
├── strategy hints
├── execution summaries
├── validation expectations
├── continuity quality
└── optional repo map
lifecycle
├── resume
├── work
└── finalize
interfaces
├── CLI
├── local MCP tools
└── generated agent instructions
```
Nothing here requires the model to become different or the agent to be conscious of the whole project forever.

The system just gives the next run a better starting point and asks the current run to leave evidence behind.

Simple.

**The hardest part is pruning**

Capturing memory is easy. The hard part is deciding what deserves to survive.

A useful continuity layer needs to keep asking:

```
Is this still current?
Was this observed or inferred?
Is this relevant to the next task?
Is this a durable decision or a run note?
Should this be loaded by default?
Should this be demoted?
Should this be cleaned?
```
This is where many memory systems become heavy.

They optimize for accumulation but continuity should optimize for reuse.

Those are different incentives.

A system that remembers everything will eventually force the agent to rediscover what matters inside the memory itself.

That is just moving the cold start to another folder.

The goal is not to build a second repository made of summaries.

*The goal is to preserve the minimum operational state required to avoid starting from zero.*

**Final thought**

I still think coding agents need memory.

But I no longer think “big memory” is the most interesting version of the problem.

For software work, the stronger primitive is continuity.

- Small. 
- Inspectable. 
- Repo-local. 
- Evidence-weighted. 
- Bounded. 
- Aware of failure. 
- Aware of active work. 
- Aware that validation matters. 
- Aware that memory can be stale. 

The future coding-agent workflow I want is not one where every agent remembers everything forever.

It is one where a new session can open a repository and immediately understand:

```
what happened
what matters now
what failed
what is unverified
what should happen next
```
Not because the model magically remembers.

Because the project does.

*If you want to know more, I’ve been exploring this through AICTX, an open-source repo-local continuity runtime for coding agents. You can check documentation or Github repository *

**The tool may change, but the architectural lesson is the part I care most about: coding agents do not only need to remember more. They need to continue better.**
