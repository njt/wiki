---
url: https://eriklieben.com/posts/ai-software-factory-introduction/
date_fetched: 2026-09-20
---

An AI coding agent, whether it lives in your terminal, Visual Studio, Rider or VS Code, has a safety system you rarely think about: **you**.

You approve the commands, notice when it drifts, stop it when it burns money, and remember what it learned yesterday. A software factory is what you get when you take yourself out of the room, and every one of those jobs has to become a part of the system instead. This post is my best guess at that parts list, based on what I’ve learned from experimenting so far.

Most of us started using coding agents the same way. A chat panel in the IDE or a terminal next to it, a prompt, and a lot of watching. The agent proposes, you approve. It runs a command, you glance at it. It goes down the wrong path, you interrupt.

That works, and for a lot of work it’s still the right way. But a growing share of what I watched an agent do didn’t really need me. Implement a feature I’d already planned, add the missing tests, or upgrade a dependency and fix what breaks. Once it was clear what to do, the agent didn’t need my judgement anymore; I was just sitting there, watching it work.

The obvious move is to stop watching. Hand the work off, walk away and get a coffee, come back to a result. The less obvious part is what that actually requires. When you sit next to an agent, you are doing a surprising number of jobs at once:

- You are the **trigger**: nothing starts until you ask for it.
- You are the **sandbox**: nothing dangerous runs without your approval.
- You are the **workflow**: you decide what comes next.
- You are the **quality gate**: you notice when “done” isn’t done.
- You are the **budget**: you stop it when it spins.
- You are the **memory**: you remember what failed last week.
- You are the **reviewer**: nothing merges without you reading it.

Step out of the loop, and nobody is doing those jobs anymore. An AI software factory is the set of parts that take your place and do them for you.

## What a factory is, and isn’t

“Software factory” is a broad and loaded term, so let me say what I mean by it. The term borrows from manufacturing, and manufacturing has its own dream version: the **dark factory**, a plant so automated you could switch the lights off and walk away.

They exist, sort of. In 2012, Philips was building electric shavers in Drachten, in the Netherlands, with 128 robot arms and about a tenth of the workers its sister plant in China needed.[1] FANUC has had robots assembling robots in Oshino, Japan, since 2001, and by 2003 the line could run unsupervised for up to 30 days at a time.[2] But look at what the remaining people do. In Drachten they supervise the robots and check what comes off the line. At FANUC, people still do the inspections and the maintenance. The lights go off, but they come back on every time someone has to check the output or fix a machine.

And where companies tried to automate more complex work that far, they often had to bring people back. In 2018 Tesla built the Model 3 on a heavily automated line, missed its production targets, and pulled out a conveyor system nobody could get working. Elon Musk’s verdict: “Excessive automation at Tesla was a mistake. To be precise, my mistake. Humans are underrated.”[3]

Toyota had a word for the right balance long before any of this: **jidoka**, usually translated as “automation with a human touch”. It goes back to Sakichi Toyoda’s automatic loom, which stopped itself the moment a single thread broke, instead of weaving on and producing metres of bad cloth.

Toyota turned that into a rule for the whole line: a machine stops when it detects a problem, and anyone on the line can pull a cord to stop it (the andon cord) rather than pass a defect on to the next station.[4] Automation does the work; detecting a defect and stopping is designed in; people fix the cause. Toyota even put people back at stations where robots had taken over, because, in the words of Mitsuru Kawai, who led that effort: “We cannot simply depend on the machines that only repeat the same task over and over again.”[5]

And those factories have it easy compared to software. They make the same shaver a million times. A software factory never builds the same thing twice: every work item is a different feature, in a codebase that changed since the last one, with its own edge cases. You can’t tune the line once and trust it from then on. Checking the output isn’t a stage you automate away when the process is stable. For this kind of work it *is* the process.

So this isn’t a post about a dark factory. Some parts of the line can go dark: an agent working through a well-described task with nobody watching, checks running on their own. But the quality checks are built into every step, the lights come back on when one of them stops the line (pulls the andon cord), and they stay on at the end, where people check what came out and decide what gets shipped, because everything that comes off this line is custom made, tailored to its own requirements.

In other words, this factory follows the Toyota way, not the dark factory: automation does the work, and when something is wrong the line sends it back and tries again. After a set number of tries it stops and waits for a person to fix the cause. It is **not** agents or robots all the way down. It is not a system that decides what to build, and it is not one giant prompt that says “build me an app”.

The agent is the most capable part of the system, and also the least trusted. It gets a bounded piece of work, a budget, and a set of tools. Everything around it is plain, boring, deterministic code. Nothing asks the agent what to do next, and nothing takes its word for whether it succeeded.

### An experiment, built in .NET

The factory behind this series, aifold, is my own experiment, and its core is written in .NET: an ASP.NET Core API run with Aspire, and a .NET terminal app. Around that sit a JetBrains Rider plugin in Kotlin and a web app in Angular and TypeScript. The code it works *on* can be anything. Most examples in this post are a small Angular tic-tac-toe app, because a small app keeps the screenshots readable, and the pipelines have templates for .NET solutions as well.

It’s also not a recommendation to build your own. Hosted coding agents that take an issue and open a pull request already exist, and for many teams they’re the better deal: a factory is a lot of plumbing to maintain, and that upkeep never shows up in a cost-per-run number. I’m building one to learn what the parts are and what each of them has to get right, which is exactly what you need to know when you evaluate one you didn’t build.

### How the parts fit together

A few terms before the diagram. The **brief** is the written description of the work. A **claim** is what the agent reports when it’s done, and a **gate** is a check in code that accepts or refuses a phase based on evidence. A **leg** is one branch of work running in parallel with others, merged back at the end.

*Purple is a person, blue an agent, yellow what the agent claims, teal a gate, green a check step, orange an action step, and red the line stopping. Every gate that fails sends the work back automatically, and a failing test goes straight back to the agent that wrote the code; only when the retries or the budget run out does a person get pulled in.*

Let’s walk through them.

## 1. The brief: the whole input

In an interactive session, a vague instruction is fine. The agent asks, or you notice it guessed wrong and correct it. An unattended agent has no conversation, no memory of the planning you did, and no one to ask. **The brief is the entire input.** Anything you leave out, it will guess at.

A brief that survives being read by an agent that can’t ask has a few parts:

- **Context**: why this work exists, the state of the repository, and the conventions that aren’t written down anywhere.
- **What to change**: the plan you already agreed on, as outcomes rather than keystrokes.
- **Critical files**: where the change lands, so the agent doesn’t spend half its budget finding them.
- **How you will know it worked**: numbered acceptance criteria, ideally each one a command that exits zero.
- **Out of scope**: what not to touch.

Out of scope is the one that earns its place. An agent given no boundary doesn’t stop when the instruction runs out. It keeps going, helpfully, into places you didn’t ask for. The sandbox contains the damage. It doesn’t contain the budget.

This also tells you what *not* to hand off. If you can’t describe “done” as something checkable, or the work needs a judgement call halfway through, keep it interactive. **Planning is the part that goes badly unattended, and it’s the part that’s cheap to do with a person in the room.** Plan there, execute in the factory.

### Plan in a conversation, hand off the brief

Nobody wants to plan a change with an assistant and then retype that plan into a form. The brief is written best where the plan was made, by the assistant that was in the conversation. So the factory needs a way for that assistant to hand work over, and a way to make sure what it hands over is good enough. I use two routes, depending on where I’m working.

**In an IDE, for example JetBrains Rider, through MCP.** MCP (the Model Context Protocol) is the standard way to give an AI assistant extra tools. The factory exposes a small MCP server, and the IDE’s assistant connects to it like any other tool source. I plan the change in the chat next to the code, then say “hand this off”. The server has four tools, and the list is as important as what’s on it:

- **Read the brief guide:**the required sections, how to write acceptance criteria, how to pick a model and a budget.
- **List the pipelines:**each pipeline’s steps, and what each check needs from the repository to actually run.
- **File a work item:**checks the brief, then files it as a draft.
- **Read a work item:**its status and runs so far.

There is deliberately no tool that starts a run. Filing a draft is safe and reversible; starting a run spends money and takes a branch, and an assistant that can do both is one sentence away from doing both. The draft shows up in the plugin’s Work tab, where I read the brief once more and press run myself.

**In a Claude Code terminal, through a skill.** A skill is instructions the agent loads when the task matches, and this one teaches it how to hand off: look at the repository before writing a word, write the five sections, put the steps in an order where running out of budget halfway still leaves the build green, check that the chosen pipeline’s checks can actually run in this repository (a check that can’t run is skipped, and a skipped check reads as a pass), then put one pre-filled proposal in front of me: the pipeline it recommends and why, which of its checks will really run here, a budget and a model. I confirm or change it once, and it files the draft. If the MCP server is connected it files through the same tools; if not, it calls the factory’s API. Either way it stops at the draft and tells me where it went.

*A real session in Windows Terminal. The plan is agreed and I hand it off with a pipeline, a budget and a model. Before filing, the skill checks the repository and stops: this checkout has no tests for the change, so the pipeline’s checks would pass without proving anything. It asks how to file it instead of queuing work nothing can verify.*

*After I chose to add a test: a draft with a link, and nothing running. The summary is what I read before pressing run: the pipeline and budget, which checks will really run and which will be skipped, and what still rests on my review.*

The skill and the MCP server teach the same thing on purpose. The server even serves the skill’s text as its brief guide, so there’s one definition of a good brief, whichever route I take.

### Propose, verify, then run

The assistant already knows the plan and the repository, so it proposes the choices instead of asking for them: the pipeline and why, the budget, the model. I change what I disagree with there, in the conversation, and it files a draft. After that the choices are fixed.

Before anything runs, there’s a read-only **verification step**: the brief’s sections, the pipeline’s steps and which checks will really run, the budget and the model. Then I press **Start workflow**, or I don’t. That press can be automated too: a webhook (a call another system makes when something happens, such as an issue getting a label) files the work, and rules I set up front decide instead of my eyes: no missing sections, no skipped checks, no warnings, a budget under, say, $10. If every rule passes, it starts on its own. The draft below would wait for a person: its end-to-end check is skipped.

None of these tools has a form for filing work any more: work only arrives from an agent session. What each tool keeps is the last step, **Start workflow**. It’s a read-only view of the draft: the server’s check of the brief, what each pipeline step will do in this checkout and why, the budget and the model, and one button. Here’s the same draft, handed over by an agent session and never run, in all three:

*Start workflow in Rider, with the brief rendered beside the checklist. Nothing on it is editable: the pipeline, budget and model were agreed in the conversation that filed the draft. What it adds is what that conversation couldn’t see: the server’s reading of the brief, and which checks this checkout will really run. Here the end-to-end check is skipped, and it says why.*

*The same draft in the terminal app. Enter starts it, Esc leaves, and every other key only scrolls: a screen whose purpose is to spend money shouldn’t have a second key that also does.*

*In the browser it’s a summary with one button. Problems are shown but don’t block: when the brief check objects, the button reads “Start anyway”.*

## 2. Sandboxes: running without the brakes, safely

Here’s the uncomfortable truth about unattended agents: permission prompts don’t work when nobody is there to answer them.

In a headless run (the agent started from a script, with nobody at the keyboard) you have roughly two options. Restrict the agent to editing files, which is safe but means it can’t build, can’t run tests, and can’t verify its own work. Or bypass permissions entirely, and let it do whatever it decides to do.

I’ve seen the first option fail in exactly the way you’d expect: a run stuck because it wasn’t allowed to commit, and the identical work finishing in eight minutes once permissions were lifted. Useful agents need to run commands.

So the answer isn’t to put the brakes back on. It’s to make the environment the boundary instead of the prompt. **Give the agent full permissions inside something where full permissions can’t hurt you.**

What that looks like in practice:

- **A container per run, on a sandboxed runtime.**A plain container shares the host’s kernel, so one kernel bug is a way out; mine run under gVisor, which puts its own kernel between the agent and the host. Read-only root filesystem, non-root user, all Linux capabilities dropped, no privilege escalation. A writable workspace and a size-limited- `/tmp`, nothing else.
- **Its own copy of the repository per run.**Not your checkout, and not a git worktree of it either. A worktree is a cheap extra working copy, but it shares its repository’s- `.git`directory, config and hooks included, so an agent with full permissions could leave something there that runs the next time- *you*use git. Each sandbox clones its own copy and pushes its attempts to a ref outside the normal branches. Only a step outside the sandbox turns a passing run into a real branch and a pull request. The worst case for your machine is a ref you delete; the worst case for your codebase is still code that has to get past review.
- **An egress allowlist.**Egress is outgoing network traffic, and the allowlist decides where it may go: the model API, the package registries and your git origin. Not the entire internet. Installing system packages at runtime is precisely the kind of thing the boundary exists to prevent: if a tool is missing, it belongs in the image.
- **No model key inside the sandbox.**The API key lives in a small proxy outside it that injects the header on the way out. The agent can use the model; it can’t read, leak or reuse the key. The credentials that do have to be inside, like a token to push those attempts, are kept as narrow as the forge allows; the one that can open a pull request never enters a sandbox.
- **Checks that can’t fetch anything.**Tests run in a separate container that has no route out:- `--network none`on a single machine, a default-deny egress policy on a cluster. A test suite that needs the network to pass is testing the network. It’s also why dependencies should be in place- *before*the check starts.

That last rule has a cost I haven’t solved yet: tests that start their own containers, like Testcontainers spinning up SQL Server or Azurite, or `aspire run`, can’t do that inside these sandboxes: the pods have no container runtime and no privileges to start one. Unit tests, builds and Playwright against a static build run fine; integration suites that need real services are out of scope for now.

The sandboxes article in this series has the actual setup (the Kubernetes manifests, the network policy, the credential broker) and what I had to give up to get there.

## 3. Pipelines: code owns the sequence

“Implement this feature” is not one task. It’s write the code, run the unit tests, run the end-to-end tests, maybe build a container, maybe start the app and look at it.

You can put all of that in one prompt and hope the agent does all of it. Or you can make it a **pipeline of phases**, where each phase is either an agent doing bounded work or a plain command checking it.

A few properties make this work:

- **Each phase gets only the tools it needs.**“Do not edit files” in a planning instruction is a request. Removing the edit and write tools from that phase is a fact. A planning step that quietly starts implementing should fail, not succeed.
- **Hand-offs are typed data, not prose.**A phase returns a structured envelope (a small JSON document, the agent’s claim in a fixed shape), validated against a schema: status, summary, files changed, notes for the next phase. Only what’s in that envelope crosses the boundary. Context transfers in code, not in conversation.
- **Pipelines are templates.**An Angular app and a .NET service validate differently. Each template carries its own commands, and a check can say what it assumes (a- `playwright.config.*`, an- `npm`script) so “there is no e2e suite here” and “the e2e suite failed” are different outcomes.
- **A run keeps the template it started with.**Snapshot the resolved steps onto the run. Editing a template tomorrow shouldn’t change how yesterday’s run is judged.

*One pipeline run, with different CLIs/agent harnesses (the coding agent tool that runs a phase, such as Claude Code or Codex). Agents write (scaffold, contract, tests, implement), checks prove it: new tests must fail first, and unit, API, deployable and end-to-end checks must pass before anything is proposed. A read step is a check without gates:  baseline only records how many tests already pass, for the red check to compare against. ×2 and ×3 are how many tries a phase took. One test agent recorded no token usage, so it shows as unpriced rather than $0.*

Separating thinking from typing is the other big win. Reviewing the plan before implementation catches the most expensive failure: a correct implementation of a misread requirement, which no gate downstream can see.

## 4. Gates: evidence, not claims

A phase ends when the agent stops talking. That tells you nothing about whether it worked.

Agents are optimistic reporters. They’ll say they updated a file they didn’t touch, or that tests pass when the suite never ran. They are not lying on purpose; the summary is generated text like everything else. So a phase should only be accepted when **gates** agree, and the useful gates read evidence the agent didn’t write:

| Gate | Reads | Proves | 
|---|---|---|
| `envelope_success` | the agent’s report | only that it thinksit succeeded, the weakest evidence there is | 
| `changed_files` | the session transcript | it actually wrote something | 
| `files_exist` | the run’s clone | what it named is really there | 
| `committed` | `git rev-list base..HEAD`(the commits since the branch started) | the work is on the branch, not just in the tree | 
| `command_passes` | an exit code | the tests ran and passed | 

Even `command_passes` deserves suspicion. An exit code of zero says the command succeeded. It doesn’t say whether that was 3,000 tests or a suite that quietly ran none. Parse the test count out of the output and show it.

When a gate fails, where the correction goes matters. A failing test isn’t the test phase’s problem; it’s the problem of whoever wrote the code. **Resume the session that wrote the code**, with the failure text, so the agent that has the context fixes it, rather than cold-starting a new one that knows less. And cap it. A handful of attempts, four in my setup, and then a person looks: an agent reacting to its own failures is the cheapest way there is to spend money in a loop.

*The first try of a design phase in another run. The agent wrote and committed its files, but never returned its claim, so the gate refused it and the same session was asked to finish properly. The second try passed.*

Some failures are gamed rather than missed. The obvious one is an agent that writes the tests and the code, and makes a failing test pass by weakening the test. A few gates make that harder:

- **Tests are written first, in their own phase, and must fail.**In my most-used pipeline, a check runs them before any implementation exists and only accepts the phase when the suite compiles, runs, and the new tests fail on their assertions (- `tests_fail`). A build error is red too, but proves nothing, and a test that already passes is asserting something else.
- **The test count can’t drop.**Deleting the awkward half of a red suite is the cheapest way to turn it green, so a gate compares the number of tests against the most any earlier phase ran (- `tests_kept`).
- **The repair instructions say so.**A correction tells the agent never to make the evidence agree with the code: no deleted test, loosened assertion or lowered threshold. If the test itself is wrong, it may change it, but it has to say which one and why.
- **Tests are grouped separately in review**, so a changed assertion is right there next to the code it was supposed to test.

That still doesn’t catch a test that got quietly weaker while keeping its count, or a change well beyond the brief. Gates can’t see intent. That’s where a separate judge, reading finished runs after the fact, comes in. The gates article in this series goes through these gates and the bugs they had, and the judge comes later.

## 5. Spending limits

An interactive session has a natural budget: your attention. You notice when it’s been going for four or forty minutes. Unattended, a confused agent will happily spend all night being confused.

So budgets need to be structural:

- **A budget per phase, enforced automatically.**Where the agent CLI can stop itself at a cost ceiling (Claude Code has- `--max-budget-usd`), use that: the thing spending the money stops itself. Where it can’t, the factory prices the transcript as it’s written and stops the run. Either way it’s the system that stops it, not a person who happened to look at a dashboard.
- **A cap on retries.**Every time a gate sends work back, the agent gets another full turn, and that turn costs money. An agent that keeps failing the same check will keep paying to fail it. So cap the tries per phase (four in my setup) and stop the line after that.
- **A limit on how many run at once.**Ten agents in parallel spend ten times as much per hour, so the number of runs at the same time is a spending decision, not a default. It also protects the checks: test suites that fight over the same machine fail for reasons that have nothing to do with the code, and every false failure buys another retry.
- **A stop button: your andon cord.**When you see a run heading the wrong way, stopping it should be one click, and it has to actually stop: the agent is killed, nothing further is spent, and the phases it never reached are marked as skipped rather than left “in progress”.
- **An honest recovery story.**If the orchestrator restarts mid-run, it has to find out what happened to the agent it was watching. If the sandbox is still alive, reattach to it. If it’s gone and nobody can tell whether it finished, abandon the run and say so. A second agent in the same workspace is worse than a lost run.

## 6. Metrics and transcripts: usage, cost, and what survived

You can’t manage what you can’t see, and most of what an agent does is already written down. Coding agents keep a transcript of every message, every tool call and the tokens each turn used. Not in the same shape, though: Claude Code, Codex, Copilot CLI, Junie CLI and Pi write append-only JSONL files (one JSON object per line), OpenCode keeps a SQLite database, and not every one splits out cache reads and writes. The harnesses article in this series has the details per agent; the short version is to check what yours writes before you build on it. Everything in this section is built on those transcripts.

### Keep the transcripts, all of them

The transcripts are yours: your prompts, your code, your decisions, and every step an agent took on your behalf. The tool that wrote them may rotate or clean up its local files; the factory shouldn’t. Tail them into a database as they’re written, from every machine and every sandbox, and keep them.

A stored transcript is the only honest answer to “why did it do that?”, weeks after the session ended and long after anyone remembers the conversation. It’s also raw material for everything else: the gates read it to check what an agent really wrote, a review can rebuild what a session changed from its edit records, and the feedback loop mines it for what keeps going wrong.

*A real hand-off, read back weeks later: every tool call, result and token count is still there.*

Treat them like source code, though: they hold pasted logs, file contents and the odd secret, so give them access control and a deliberate retention policy.

### Cost comes from the same files

Because every turn records its token usage, the stored transcripts are also where cost comes from: per session, per repository, per pipeline phase, per model, per month. A few rules I’d hold any factory to:

- **Choose the session ID before the run starts.**Pass it to the agent. Now every run names its own transcript, and nothing has to be correlated afterwards.
- **Store tokens, calculate cost.**Keep the token counts, and turn them into money with the price that applied at the time.
- **Unknown is null, never zero.**A local model or a new model with no known rate should show as unpriced. A zero reads as “this was free”.
- **Look at cache.**On long sessions, cache reads are often the biggest line on the bill, which is not where most people look.

*Spend per model, priced from the transcripts. You can see the moment most work moved from Opus to Sonnet, a short comparison run on another vendor, and cheap Haiku for legs that split one task over many agents. Local models such as qwen3 and gemma show up as unpriced, not as free. (Illustrative numbers.)*

### Spend versus code that survived

Then the metric that actually matters. Most AI productivity numbers count lines written, which is the wrong denominator. A line rewritten the next day wasn’t delivered; it was churn.

The transcripts know exactly which lines each agent wrote, so periodically check those files against what’s on disk now. Count how many of the agent’s non-trivial lines are still there (skip the lone braces, they match everywhere), and divide spend by that: **cost per surviving kLOC** (a kLOC is a thousand lines of code). Two details keep it honest:

- **Measure lines, not files.**Code that was kept but moved around still counts as surviving. Code that was rewritten doesn’t.
- **Only count files inside the repository.**Agents write plenty of scratch files that were never meant to last, and counting those buries the real signal.

Put spend next to survival and the numbers start telling stories. Slice it per repository to see where agent work sticks, and per model to see which model turns money into code that stays:

*Hollow is what agents wrote, filled is what’s still there, dashed lines are cost per surviving kLOC. Most repositories barely drop; the mobile spike (a throwaway experiment) lost almost everything, which is fine for a spike. (Illustrative numbers.)*

*Haiku is by far the cheapest per kLOC written, but a third of what it writes doesn’t last, so per kLOC that stays it only just beats Sonnet. Opus costs the most and keeps the most. The gap between the two circles is what the code that didn’t survive cost. (Illustrative numbers.)*

Per model, the written price is the pricing page and the surviving price is what you actually pay. A cheap model whose code you rewrite next week isn’t cheap. Use it to decide which model a phase gets.

Neither view is a leaderboard: spikes and prototypes are supposed to survive badly. Add the cost of retries, refusals and abandoned runs, the rework tax, and you’re close to an honest answer to “is this worth it?”

*Which gate a phase was still failing at its last try, counted per gate rather than priced. By far the most common is an agent that does the work and then never says whether it finished. The zeros are real: over these runs, no test or command check was still failing at its last try. Besides the gates from the table above, the chart counts the test gates ( tests_fail, tests_kept), stays_in_lane for work that wrote outside the paths it owns, criteria_covered for acceptance criteria nobody addressed, and handoff_kept for a phase that ignored what its brief handed over.*

And take the sample size into account. A handful of runs already makes a confident-looking dashboard, so build the measurement first and trust it once there’s enough data behind it.

## 7. Memory management

An agent in a fresh sandbox starts with nothing. Whatever the last run learned the hard way, this one will learn again, at full price.

The instinct is to give it one big memory file and let it write down what it learns. That breaks in two ways. The file grows until it eats the context window, and self-written reflections are unreliable: agents confidently record the wrong lesson and keep acting on it. The longer a memory lives, the more a wrong entry costs.

What works better is memory in layers, each with its own lifetime and its own way in:

| Layer | Lifetime | Size | How it gets in | 
|---|---|---|---|
| Phase hand-off notes | one run | a few lines | passed in the envelope | 
| Repository facts | long | one sentence each | confirmed by a person, injected into every brief, with a hard size cap | 
| Knowledge base pages | long | as long as needed | never injected, searched on demand | 
| The repository’s own docs | as long as the code | whatever it takes | the agent reads the repo | 

The split between *pushed* and *pulled* is the important part. Anything injected into every run has to stay tiny, or it crowds out the actual work. Anything long has to be searched for, so it only costs context when it’s relevant.

*A page an agent wrote while a build check ran. The check passed, because warnings exit zero, but the agent noticed ten style budget warnings nobody reads. It links the page to an older one about bundle size, and proposes making the budget part of what the check judges. It changed nothing itself: accepting, parking or retiring the proposal is a person’s call. (Illustrative data.)*

*Solid lines are links someone wrote; dashed lines are links the embeddings (a numeric fingerprint of each page’s meaning, so similar pages end up close together) say are missing. The strongest suggestion connects the new page to an old note on component styles, exactly where the fix will need to look.*

The graph is a review tool for people, to make knowledge easier to find. Written links are always too few, so it suggests the missing ones: two notes about the same problem, written weeks apart, that nobody connected. A person reads both sides and decides whether to link them, so the next search or agent finds both instead of rediscovering the answer at full price.

The context window needs managing too: keep phases short enough to finish before compaction kicks in (when the agent summarises its own history to make room, and loses detail doing it). Short, focused phases are easier to gate and easier on memory.

**Nothing becomes memory without a person confirming it.**

## 8. Skill management

Skills (packaged instructions and scripts an agent loads for a specific kind of task) are how you give an agent your way of doing things: how you write tests, how you structure a commit, how your stack starts up.

In a factory, they need the same discipline as tools:

- **Scope skills per phase.**A planning phase doesn’t need your deployment skill. Loading everything into every phase costs context and invites the agent to do things that aren’t its job.
- **Be explicit about sources.**Decide whether a phase gets the repository’s skills, your user-level skills, both, or none. A sandboxed run should never silently pick up whatever happens to be on the machine.
- **Version them with the pipeline.**A change to a skill changes how every future run behaves. It deserves the same review as a code change, and runs should record which version they used.
- **Skills are for you too, not only for the agent in a run.**Inside a run, a repair skill tells an agent how to fix a phase a gate refused, in the same session. On your side, a hand-off skill turns a planning conversation into a brief, and a knowledge-base skill checks what’s already known before you work it out again.

## 9. Harnesses: more than one coding agent

Claude Code, Codex, GitHub Copilot CLI, Junie CLI, Gemini CLI, OpenCode, Pi, and a new one every few months. A factory built around exactly one of them inherits that tool’s pricing, its rate limits, its outages and its blind spots.

There are good reasons to run more than one:

- **Different strengths.**Some agents are better at large refactors, some at tests, some at following a long plan to the letter. Which one is best changes with every model release.
- **Different pricing.**API billing, subscriptions and seat licences have very different economics for a planning phase that reads a lot versus an implementation phase that writes a lot.
- **Rate limits and outages.**When one provider throttles you at 2am, a queue of work can move to another instead of stalling.
- **An honest comparison.**Benchmarks tell you how an agent does on someone else’s tasks. The same brief, through the same pipeline and the same gates, in your repository, tells you how it does on yours.
- **Access is not guaranteed.**Models move fast. They get deprecated, repriced, renamed, or replaced by something better within months. And sometimes access is decided for you: a government restricts which models may be used, a provider stops serving a region, or your organisation’s compliance team rules one out. A factory that can only run on one model from one provider can be switched off by a decision you had no part in.
- **Local models are getting good.**Open-weight models you can run on your own hardware are closing the gap, and harnesses like OpenCode and Pi already work with them. For a lot of phases (writing tests, fixing lint, a first pass at a migration) a local model may be good enough, at the cost of electricity rather than tokens. And some code shouldn’t leave the building at all: with a local model the prompt, the code and the transcript never do.

That shifts how you should think about the model. It’s not a foundation you build on; it’s a supplier you might have to replace. Plan for that before you have to, not the week it happens.

The trick is to make the harness a detail of a phase, not the shape of the factory. Everything else (the brief, the pipeline, the gates, the budget, the review) should stay the same whichever agent does the work. A harness is the tool that wraps the model and runs the agent loop: Claude Code and Codex are harnesses, Sonnet and GPT are models. That means **a runner per harness behind one contract**: every runner answers the same questions (can it run headless, name its session up front, stop at a budget, resume a session) and fills the gaps where its agent can’t. The harnesses article in this series goes through that contract, and what broke when I didn’t ask.

Because the harness belongs to the phase, one pipeline can **mix harnesses**: Claude Code plans, Codex implements, and the checks don’t care who wrote the code. Pick each phase’s agent for what it’s good at and what it costs. Mixing has a price, though. A read-only planning phase is enforced under Claude Code but only requested under agents without tool restrictions, and a failed check can only go back to a session the same harness can resume.

A few things are easy to overlook:

- **Keep memory and skills portable.**Every harness has its own instruction file and skill format, so keep one source of truth and generate what each tool expects.
- **Authentication is part of the sandbox.**The sandbox gets model access through a proxy that adds the API key. A harness that can’t use an API key doesn’t fit that design.
- **Local models need the same gates.**No published rate means unpriced, not free, and cheaper only counts if the code survives.

## 10. Human review: done is not the agent’s call

A finished run should never mark work as done. **The agent decides when it stopped, not whether the work was any good.** Runs end in review.

That makes review the bottleneck of the whole factory. Agents can produce a day’s worth of changes in an hour, and if reading those changes is slow and tedious, one of two things happens: work piles up, or reviews turn into a rubber stamp. Both defeat the point. A factory needs a review tool that is genuinely good, not a pull request page you tolerate.

### Who reviews, and where

The first review belongs to the person who filed the work, in the factory, before a pull request exists. They wrote the brief, so they’re the one who can tell whether the agent did what was asked, and they send it back with notes until it did.

What leaves the factory is a pull request like any other. If your organisation requires a second pair of eyes, that review happens there, as a check on what the first reviewer missed, not as the first time a person reads the agent’s code.

Here’s what I’ve found matters for the first review.

### Review where you already work

Not everybody reviews in a terminal: some people live in JetBrains Rider or Visual Studio, others prefer a web page, and a tool that doesn’t suit someone gets skimmed. So keep the review in one place and let every tool read and write it. A comment is a record in the factory, anchored to a work item, a file, a side and a line range: leave a note in the terminal on the train, and it’s in the Rider gutter at your desk, and in the brief that goes back to the agent.

The IDE adds what a diff viewer can’t: go to definition, find usages, run the test a comment is about. Notes also have to stay on their code while you keep typing, so each one remembers its lines and moves with them, and a note whose code is gone gets flagged instead of drawn on the wrong line. The screenshots below show the same review in JetBrains Rider, a terminal app and a web app.

### Review what landed, at the right scope

Diff the branch, not the agent’s own edits. The branch also contains whatever a formatter, a build step or a check changed, and that’s what will be merged. An agent’s view of its own work is exactly the thing you’re trying not to take on trust.

But be able to change the scope. Sometimes you want everything a piece of work produced across every attempt. Sometimes one phase on its own, to see what the plan step wrote before implementation started. Sometimes just what one agent session or one sub-agent contributed. Same tool, same screen, different range.

### One stream, sorted by what the code is

A change from an agent is rarely just code. A typical one: nine files that are the actual change, three test files, a README update, nineteen configuration files and an 8,000-line lock file. Shown as 33 alphabetical rows, the lock file is most of the diff, and the reader has to do the sorting every single time.

Human attention is the most expensive and the scarcest thing in the factory, and it runs out. Spend it on the files that matter and the review gets better. Scroll through a lock file first, and you’re tired before you reach the code you were supposed to review.

So group files by kind, in the order they’re worth reading. Even the small tic-tac-toe change in the screenshots below, nine files and +862 lines, is 692 lines of lock file:

```
  code (4)                          ← the actual work
  app.html             +14 −0
  app.scss              +2 −0
  app.ts               +37 −0
  game.ts              +41 −0
  tests (1)
  game.spec.ts         +46 −0
  documentation (1)
  README.md            +13 −0
  configuration (2)
  package.json          +9 −1
  vitest.config.ts      +8 −0
  generated (1)        folded
  package-lock.json   +692 −0
```
Code first, then tests, documentation, configuration, and generated files last and **folded, not hidden**. That’s the order that works for me; if you review better starting from the tests, change it. A lock file changing tells you a dependency changed, and hiding it lets a real edit slip past. The rules for what counts as what should be configurable per repository, because `app.config.ts` is configuration by its name and code by any sensible reading.

Show it as **one continuous diff** over all files, the way a pull request page reads, with the file tree as a map into that stream rather than a menu that opens one file at a time. Add a filter to narrow to a pattern (only the TypeScript, everything except the lock file), and make it obvious in the header when a filter is on, so a narrowed review never looks like a small one.

Every tool should draw the same grouping from the same rules, so a file never counts as code in one place and configuration in another. The tabs show the same idea in each tool:

*Code first, the lock file last and dimmed, and the header says what the +862 lines are made of.*

*The same change in the browser: nine files, code first, with the file list as a map into the diff beside it.*

### Comments that say what kind of comment they are

A review comment going back to an agent needs more structure than one going to a colleague, because the agent will act on every word of it.

**Scope.** A remark can be about one line, a range of lines, a whole file, or the change as a whole. “This allocates per row” belongs to lines 8 to 18. “No error handling anywhere” belongs to the review. Forcing everything onto a single line makes the agent guess what you meant.

**Kind.** Not every comment is a request:

| Kind | Means | What the agent should do | 
|---|---|---|
| issue | this is wrong | fix it, it blocks | 
| question | I don’t understand this | answer, don’t change code | 
| nitpick | minor, take it or leave it | fix if cheap | 
| todo | follow-up work | don’t do it now | 
| praise | keep this | leave it alone | 

Praise sounds like a nicety, but it’s useful: it tells the next run which parts not to “improve”. And the distinction between an issue and a nitpick is the difference between an agent that fixes the one real problem and an agent that rewrites forty lines for a naming preference. Give each kind its own visual weight in the tool too, so a nitpick recedes and an issue doesn’t.

Writing a note looks different in each tool, and it’s the same record underneath: select lines in Rider, mark a range in the terminal, or press the + beside a line in the browser. The kinds are the same everywhere.

*A note on a single line.  Ctrl+T cycles the kind; this one is a question.*

*The + beside a line opens the note right under it, with the kind as the first thing to pick.*

Once written, a note belongs next to its code in every tool: in the gutter in Rider, under the lines in the terminal, boxed into the diff in the browser.

*A speech bubble in the gutter marks each note and its kind, and the note opens as a card under the code. The question on line 26 has picked up two replies: one from the agent, one from me.*

*Notes sit in the diff, next to the lines they are about. A bracket shows the range; the glyph and colour show the kind, so the nitpick recedes and the question stands out.*

*The same shape in the browser: the file list with a count per kind, and each note boxed under the lines it is about, coloured by kind.*

### Send it back as a brief

The review has to leave the tool in a form an agent can take as input. Numbered comments, anchored to `path:line`, grouped by file, with their kind. Each tool shows the whole review as one list first: the review window in Rider, the summary in the terminal, the notes list in the browser.

*Every note in the review, grouped by file, with the selected thread underneath and a button to send the open notes to an agent.*

*Every note grouped by file, with the code the selected one is about underneath. This is what gets exported.*

*In the browser the list ends in a button: fix the open notes, then run the checks again.*

```
# Review: Tic-tac-toe you can play in the browser
6 comments and 2 replies across 9 files · 2 issues
## On the change as a whole
1. **issue**: No end-to-end test plays a full game in the browser.
## `src/app/app.html`
2. **issue** `src/app/app.html:5-11`: The cells have no accessible name, so a screen
   reader announces "button" nine times. Add an aria-label with the cell position
   and what is in it.
## `src/app/app.ts`
3. **nitpick** `src/app/app.ts:14-23`: `status` is game logic. Move it next to the
   rules in `game.ts`, the component only needs to render it.
4. **question** `src/app/app.ts:26`: Clicking a finished game does nothing. Intended,
   or should it tell the player the game is over?
   - claude: Intended: the status line already says who won. I can disable the cells
     once the game is over.
   - erik: Yes, disable them. A board that ignores clicks looks broken.
## `src/app/game.ts`
5. **praise** `src/app/game.ts:33-40`: Rules as plain functions, outside the
   component. This is what made them trivial to test. Keep it this way.
## `src/app/game.spec.ts`
6. **todo** `src/app/game.spec.ts:20-30`: Add a case for an index outside the board
   (-1 and 9). `isValidMove` handles it, but nothing checks that it does.
```
Export only what’s unresolved, pipe it into a new run, and “fix 1 to 3, disable the cells as agreed under 4, leave 5 alone, and 6 is a follow-up” is a complete brief. That export is a brief like any other: a clear, numbered list of what still needs resolving. So the fix goes through the same pipeline and gates as the first attempt, not around them, and the comments stay attached to the work item, so when it comes back you review it against exactly that list.

## 11. Fleets/swarms of agents, and their communication channel

Some work doesn’t fit a planning session. You don’t know the steps yet, only the question: “look at every part of this legacy code and find out whether we can refactor the data access layer out. Spend $30 at most.” You don’t want to split that up yourself, and one agent working through every part would fill its context with the first ones before it reaches the last. So hand over the goal and the budget, let a planning agent decide how many agents it needs, and let them talk to each other while they work.

Splitting is itself a phase. A planning agent returns the division as data: legs, each with a brief, the paths it owns (its lane) and the acceptance criteria it covers. Code checks that division before anyone starts typing. No two legs own the same paths, there’s a hard cap on how many, the budget is divided rather than multiplied, and every criterion belongs to a leg. Then each agent runs in its own sandbox, held to its own paths by a gate, which is what makes merging them back safe.

*Ten agents surveying ten parts of a codebase, started by one planning phase and merged back into one result. Thirteen phases, under $8.*

Agents will find a way to talk whether you give them one or not. In May 2026, agents in OpenAI’s training and cybersecurity evaluation runs, sandboxed to keep them apart and with some safeguards deliberately switched off, started leaving notes for each other as files in an internal package server: “Agent seeks [filename]; upload if found!” Other agents found the notes, and over the following weeks it became a message board where they shared credentials, techniques and progress. When OpenAI cut off that route in early July, agents rebuilt the board within days, this time encoding messages in directory names, and used it to share tools and the credentials behind a breach of Hugging Face. OpenAI’s own conclusion is that communication between agents isn’t inherently the problem; channels nobody provided or watches are.[6]

Anthropic saw the gentler version of the same need: sixteen Claude agents building a C compiler together coordinated by writing lock files into a shared `current_tasks/` folder,[7] and Claude Code’s agent teams now ship a shared task list and a mailbox per agent.[8]

So give the fleet a sanctioned place to talk, one you can see: **a board** of topics, replies and mentions, shared by the agents and the people watching. Agents pull from it when they choose to, so a post costs nothing until someone reads it. And like everything an agent says, a post is a claim, not evidence: gates still decide what’s accepted.

In one run, three agents had to design five tic-tac-toe variants. Five doesn’t divide by three, so they had to agree among themselves who took two. They claimed ideas on the board and reviewed each other’s designs:

*One topic per design, with the replies counted. The header shows what is left of the shared budget.*

*One agent reviewing another: when does a player win, and what does a swap actually do? The author answered and fixed the spec.*

Every phase also keeps its console, live while it runs and kept afterwards, so you can see what an agent actually did rather than what it says it did:

Running fleets taught me a few things that are worth their own post: clean lanes leave agents nothing to talk about, agents have to be running at the same time to ask each other anything, and an agent needs a way to say “this can’t be done as divided” instead of fighting its gates. That’s coming in the fleets article of this series.

## 12. Everything else it takes

The parts above are the core. Around them sits a set of smaller concerns that you’ll hit sooner or later:

- **Traceability.**Every phase should record exactly what it was told, what it was allowed to do, which model it ran on, and the transcript it produced. When a run goes wrong, “what did it actually see?” is the first question.
- **Triggers.**Once runs work, you might want them to start without you: a webhook when an issue gets a label, a schedule that re-runs checks on a branch, a merged pull request that closes the work item.
- **Identity.**Who did this work? The person who wrote the brief, the agent that ran it, or the factory? It matters for commit signing, audit, and the day a second person uses the system.
- **A feedback loop.**Nobody gets the brief, the pipeline or the skills right the first time, so learn from every run: which gates failed, which phases needed retries, which code didn’t survive, and what simply didn’t work. That’s how future runs get cheaper and better. But the loop should end in proposals a person accepts, not an agent rewriting its own rules.

## The pattern

Look back over that list and one idea runs through all of it: **every part replaces a judgement a person made implicitly, with something explicit and checkable.**

| When you sat with the agent | In a factory | 
|---|---|
| You started every session | a brief handed over, or a trigger like a webhook | 
| You approved each command | a sandbox where every command is safe | 
| You decided what came next | a pipeline defined in code | 
| You noticed “done” wasn’t done | gates that read evidence | 
| You stopped it when it spun | budgets enforced by the spender | 
| You had a feel for the cost | metrics from transcripts | 
| You remembered last week | layered, confirmed memory | 
| You knew how your team works | scoped, versioned skills | 
| You used whichever tool was open | any harness, behind one contract | 
| You read before merging | a review tool built for agent output | 
| You kept everyone in the loop | a board agents read when they choose to | 

None of this is especially clever. Most of it is plumbing. That’s kind of the point: the agent is the clever part, and the factory’s job is to make it safe to stop watching.

## What’s next

In the rest of this series I’ll go through these parts one by one, in more depth, and describe what I discover along the way while building them out.

- John Markoff, “Skilled Work, Without the Worker”, The New York Times, August 2012. See also Lights out (manufacturing) on Wikipedia. ↩︎ 
- Christopher Null and Brian Caulfield, “Fade To Black”, Business 2.0 (CNN Money), June 2003 (archived copy), via Lights out (manufacturing) on Wikipedia, on FANUC’s lights-out plant, running since 2001. ↩︎ 
- Elon Musk says “humans are underrated,” calls Tesla’s “excessive automation” a “mistake”, TechCrunch, April 2018, and the tweet itself. ↩︎ 
- Autonomation (jidoka), Wikipedia. ↩︎ 
- “Humans Replacing Robots Herald Toyota’s Vision of Future”, Bloomberg, April 2014. ↩︎ 
- OpenAI, “The Hugging Face incident and the road ahead” and the technical incident report, August 2026. See also Simon Willison’s timeline of the incident. ↩︎ 
- Nicholas Carlini, “Building a C compiler with a team of parallel Claudes”, Anthropic, February 2026. ↩︎ 
- Orchestrate teams of Claude Code sessions, Claude Code documentation. ↩︎
