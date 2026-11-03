---
url: https://coldtake.dev/blog/bondsmith-hard-bonds-for-soft-claims
date_fetched: 2026-11-03
---

"All tests pass." I have read that sentence at the end of an agent session more times than
I can count, and more than once it was false. The tests weren't run, or they were run before
the last edit, or they were never written.

Bondsmith is the workflow engine I built around one rule: no claim counts without a proof
bonded to it. I still want the model to do what it is good at, which is generating code.
But whatever it tells me about that code afterwards, I check with something which is not an
LLM.

Where the name comes from

Brandon Sanderson's The Way of Kings, the first book of the Stormlight Archive. The
Bondsmiths are one of the ten orders of the Knights Radiant, and the rarest: there are never
more than three of them. The other orders fight, heal or travel. A Bondsmith's power is
connection, binding things and people together. Highly recommend this one.

Work is admitted only when its entry conditions hold, and it counts as done only when its
exit conditions are shown to hold against real evidence. The LLM is free inside one step.
Whatever it says about that step afterwards is a soft claim, and a soft claim needs a hard
bond. Let me show you.

Six requirements decide whether a system for agentic development works for me.

1. Deterministic proof. I don't want to take the LLM's word that an action happened. For
every action I care about, a tool runs a check and writes the result by deterministic
action.

2. No single harness. I don't want to be chained to one LLM provider, or to one agentic
coding harness like Claude Code or Codex. I want to be able to use different ones. Models
change every few months and the pricing and limits change with them, so I don't want to
rebuild my whole workflow every time I switch.

3. Generic workflows. There is no one generic approach, one SDD definition or one flow
we can use for every implementation, every feature delivery and every piece of tech debt. A
new feature needs a spec and a breakdown into tasks. A bug fix needs neither. I should be
able to define my own workflows and change them without touching the engine that runs them.

4. A CLI. I want to run the commands for the different actions from the terminal. I can
repeat a command, I can put it in a script, and it was the simplest thing to build first.
Everything else can be built on top of it.

5. Observability. I want to see anything and everything. I want to see agent logs. I
want to see the effects of the deterministic gates. I want to see the definition of each
task. Everything under my control. When a run goes wrong I need to know what the agent
read, what it changed and which check stopped it, without asking the agent.

6. One place to look. I want a UI which presents the different features in one place,
so I can easily check where I am with a given implementation. When I have a few features
running in a few repos at the same time, I lose track pretty quickly, and I don't want to
open five terminals to find out where I was.

Not all six are equally important to me. The first one is the reason I built Bondsmith at
all, and the other five are there to make it hold no matter what kind of work I do or which
model I use.

Bondsmith is a Rust workspace: one core library, a thin CLI, and a
read-only UI. The crate layout is on the Bondsmith project page.
Here I care about who does what during a run.

A Worker is a disposable LLM session spawned for exactly one step of work, which then
exits. A Gate is a deterministic check, a test or a lint run, whose verdict comes from
the tool. The Driver is the state machine in the middle: it spawns Workers, runs Gates,
and is the only thing that writes state.

Each repo configures its own set of Gates, such as lint, type-check and the test suite. That
set is the repo's gate pool, and a Flow can ask for the whole pool to run at once.

There is no arrow from the Worker to state in the figure. The Worker edits files and makes
commits, that's all it can do, and nobody asks it whether it passed or failed.

Every step in Bondsmith declares Postconditions: assertions over the state of the world
after the step has run. The Driver gathers the facts itself, from disk, from git and from
Gate results, and evaluates each Postcondition in plain Rust code. The vocabulary is closed.
A few of the verbs:

exit_zero. The Gate's command exited with code 0.

sha_is_ancestor. One commit is in the history of another.

non_empty: spec.md. The file exists and has content.

gate_pool_pass. Every Gate in the gate pool passed.

A missing fact fails closed: if the Driver can't find the evidence a verb needs, the
Postcondition fails.

A Postcondition either passes or hard-blocks the run, there is nothing in between.

This way "the tests were run" is exit_zero: role:test, checked against the exit code of a
test command which the Driver ran itself.

Nothing in a workflow definition knows which harness runs it, or which model. The contracts
and the prompts carry no harness-specific text, and Workers are launched through one port in
the core. Claude Code or Codex, or anything else with a headless mode, can sit behind that
port.

Today one launcher sits there: Claude Code. Pi is the next target on the
Roadmap.

A workflow in Bondsmith is called a Flow: a named, ordered chain of Phases, each
with its own Steps and Postconditions. A Flow is plain data, JSON contracts and Markdown
prompts in a directory, read from disk at run time. Nothing is compiled into the binary, and
no Flow is built in. I can copy a Flow, drop a Phase, add a Gate, and the next run uses it.

Every run leaves three things on disk. A state file, which is the single source of truth.
An append-only event log, one JSON line per event: step started, Gate finished,
Postconditions checked, step advanced or held. And for every headless Worker, a Session log
with its raw output.

For a headless Worker the raw log is a stream of JSON, so Bondsmith also derives a Session
transcript from it: every file the Worker read, every edit, every command, in order, with
the model, the tokens and the duration.

The trail on disk is complete. The Bondsmith UI is a local, read-only page over the same
files. Its first tab lists every feature across my projects with its current status.

Open a feature and you see where the implementation is: the Phase it is in, the Task graph,
and the progress of every step for every Task. Open a step and you are in that Worker's
Session transcript.

I use two Flows. feature is the main one, of course: it takes a feature from an
interview to a set of merged pull requests. address-issue takes one GitHub issue to one
pull request. A third, file-fix, is address-issue with a diagnosis Phase in front.

The Phases are different because the work is different. A feature needs an exploration, a
spec and a graph of Tasks. An issue already says what is wrong, so a plan is enough and we
can go straight to code. I described the Phases of the feature flow in
Bounded Chaos, back when it was a pile of bash and slash commands.
The Phases are mostly the same as back then. The difference is that now each of them is a
typed contract, enforced by the engine.

Both Flows keep one Phase: a test-first implement. In the feature Flow it runs nine Steps
for every Task.

Here is a real run where one Postcondition of the second test Gate didn't hold:

Going through the checks from the bottom up:

gate_failed_at: red_test_sha,role:test found true. The Driver checked out the commit
where the Worker claimed to have written a failing test, ran the tests there, and they
failed.

sha_is_ancestor: red_test_sha,HEAD found true. That commit is in the history of the
current code. It wasn't written afterwards and slipped in.

gt: test_count,baseline found 1711,1708. The suite had 1708 tests before the Task and
has 1711 now. Tests were added, and I know how many.

sha_recorded found true. The red commit exists at all.

exit_zero: role:test required 0 found 1. And still one test fails right now.

So four checks out of five hold, and it is still not enough. The step is held until the
failing test passes.

The review Steps produce Findings: one issue each, with a type, a severity, a file and a
proposed fix. Each reviewing agent writes its Findings into a report, one per Task. Ruling
on them by hand, I kept making the same ruling again and again. So in the feature Flow the
system learns from its mistakes, and from my decisions.

The first mechanism is Triage rules. Each repo carries its own, ordered,
first match wins. This is a real one:

An accepted Finding gets fixed, without asking me first. A Worker groups related Findings
into Review units and applies the rules. The Driver doesn't trust the Worker's triage
either: it re-derives every claimed decision from the rules, in code, and a mismatch
hard-blocks. There is also a floor no rule can cross. A rule may never
decide a security Finding, or anything above medium severity. Those always come to me.

The second mechanism is the learn-from-reports Phase. The reports are the ones the review
Steps wrote during implement. The Phase mines their Findings and my decisions on them for
patterns, and puts each candidate in front of me to accept, reject or
edit. A decision I keep repeating is proposed as a new Triage rule, in a pull request
against that repo. A lesson general enough to reuse is written as a rule into the
Knowledge base.

The mechanism for feeding these rules back is fully there. What I'm working on now is how
to use it in the different implementation and review Phases, so the next Task starts with
the rule already in place.

There is a lot, lot, lot still to cover. The system learns from the codebase, but the
system also needs to evolve, and the observability I built for myself is the best source for
that. Every Worker run already leaves a Session log and its cost behind. Analyzing them
tells me why a single step went wrong, and where model routing can make the whole Flow
cost-effective. Reading them all should tell me what to change in Bondsmith.

If you take one thing from this post, you don't need Bondsmith for it: pick one action your
agent reports on, and replace the report with a check a program runs.

"All tests pass" becomes the exit code of the test command, run by a script after the
session ends. "I committed the fix" becomes a git log on the branch. "The report is written"
becomes a file that exists and is not empty.

Start with the claim which burned you the most. For me it was the tests. One check is enough
to feel the difference, because the agent can say whatever it wants and you don't have to
believe it anymore.

Bondsmith is the same habit, just applied to every step. Start with one check and see for
yourself.

Bondsmith is going open source

I'm opening the Bondsmith repository in the upcoming weeks. Subscribe to not miss the
day!

Deep DiveAIWorkflow

Thanks for reading

More like it are on the way - subscribe to catch them.
