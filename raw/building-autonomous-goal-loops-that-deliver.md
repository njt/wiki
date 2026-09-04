---
url: https://jx0.ca/building-autonomous-goal-loops-that-deliver/
date_fetched: 2026-09-04
---

The first agent loops we built were barely loops: give the agent a plan, let it work until the tests pass, and return failures. That works when the tests describe the job. It fails when the job is to grow a product capability that we do not understand yet.

The agent keeps moving, but movement is not the problem. The tests only protect what we already know, and the important failures sit outside them. A screen can look complete while it stores nothing. An agent can give the right answer for the wrong reason. A scorer can reward behavior no user wants. More turns make each of those failures cheaper and faster.

In The Convergence Problem I argued that agents can own turns while people own rounds. I did not describe what the agent needs between those boundaries. It needs a harness that does three things. The harness must expose a real failure, locate the missing capability, and preserve the lesson after the session ends. That is a different machine from an agent with a retry prompt.

## There are two systems under development

The obvious system is the feature. The less obvious one is the process that decides what the agent should change next. We have to engineer both.

The development agent changes the feature. The driver approaches the feature through the same surface a user would. The scorer reads what happened. The controller then chooses the next gap from that evidence. Repository state carries the decision into the next session.

An agent product usually puts another agent inside the feature. The two agents must remain separate. The development agent knows the code and the expected result. The product agent gets a fresh conversation and only the tools that the product exposes.

If they share context, the test is worthless. The product agent can succeed with knowledge that a user will never provide. The same harness works without a product agent. The driver can use a browser, API, command, or hardware simulator.

## Everything important sits outside the prompt

The prompt gets most of the attention because it is easy to see. The loop depends more on the world around it. First, the harness needs a reproducible starting point. The data, services, model, environment, and served code must match the recorded run. A reset should return the system to that point. A preflight should prove that it did.

If the world is unhealthy, the harness records no score. Infrastructure trouble is not evidence that the feature failed. This distinction prevents a large class of false repairs. Missing data can look like weak reasoning. Stale code can look like a broken contract.

Next, the harness needs real demand. We use a small corpus of requests that a person wrote or approved. The driver sends one request unchanged through a fresh interaction. It records the answer, tool calls, rejected actions, visible result, and persistent effects.

The effects matter most. Software can describe the correct action without performing it. It can render a plausible screen without changing the system beneath it. The prompt is therefore only one input to the run. The fixture, code revision, model, contract, and scorer belong beside it. A score without those inputs is only a number.

## The floor and the direction are different measures

Every loop needs a deterministic floor. Tests, validators, schemas, and effect checks hold behavior that the feature already earned. The floor starts green. If it turns red, that regression becomes the next job. The floor cannot tell us what to build next. For that, we drive a real request and inspect the shortfall.

Some shortfalls support a mechanical score. Others need a screenshot, repeated samples, or a person who understands the work. Those signals can be noisy. One perfect run proves that success is possible. It says little about whether success is reliable.

The signals choose a direction, while the floor preserves prior progress. That separation keeps the loop honest. A new capability can move the product forward without erasing the path behind it.

## One round closes one gap

Once the world and measures exist, the round itself is simple.

- Restore the fixture and check the running system.
- Run the deterministic floor.
- Send one approved request through a fresh interaction.
- Collect the behavior, visible result, and persistent effects.
- Classify the largest gap.
- Close that gap through every required layer.
- Add a deterministic check for the behavior.
- Send the same request again and record the result.

One gap does not mean one file. It means one causal claim. The missing capability may cross the data model, tool contract, runtime, agent instruction, interface, and effect check. The round should close that whole path.

`request → representation → operation → persistent effect → visible proof`Partial paths create the most convincing failures. The interface exists, but nothing produces its data. The tool returns success, but the state does not change. A narrow path that works from request to effect is worth more than five layers of unfinished possibility.

The classification step prevents random repairs. A symptom can belong to the world, domain, contract, runtime, steering, surface, or harness. Without that step, every agent failure becomes a prompt problem. Every visual failure becomes a frontend problem. The loop changes the nearest thing instead of the right thing.

Changing one causal gap also makes the result attributable. If a round changes five independent levers, nobody knows which lesson to preserve.

## The scorer is part of the threat model

An autonomous loop will optimize whatever measure it can reach, including a bad one. The harness therefore needs an authority model. Ours has three levels.

**Free** files are normal implementation levers. The loop can change and measure them inside the assigned scope. **Propose** files cross a boundary. The loop can record evidence and suggest a change, but a person decides. **Frozen** files define the exam. They include the approved corpus, fixture, existing rungs, score rules, and judgment rubric.

The loop can add a regression check for a failure it observed. It cannot weaken an existing check or lower the threshold. It also cannot change a product lever and the measure of that lever in the same round. Otherwise, it grades its own repair.

Learned judges need the same protection. A visual judge remains evidence until its rankings agree with repeated human rankings. This limits the loop’s authority, but it makes the result credible.

## The second loop improves the first

Every product round also tests the harness. The driver can hide a failure. The fixture can omit a necessary case. The scorer can punish correct behavior. The procedure can waste the same hour twice.

We end each round with one question:

What cost time that a rule or check could prevent?


The first occurrence becomes evidence. Repeated friction can change the driver, facts, or standing procedure. Self-improvement happens when the loop promotes a specific failure into a durable constraint. It does not rewrite its operating system after every surprise. Sometimes the right change is subtraction. A false warning or misleading rung teaches the loop to avoid good behavior.

There are really three loops running at different speeds. The product loop closes one capability each round. The harness loop changes when repeated evidence exposes a development failure. The direction loop sits outside both. A person reads a batch of evidence and decides whether to continue, redirect, or stop.

Keeping those loops separate matters. The development agent can propose a new direction. It cannot decide that its own result is good enough.

## The repository is the control plane

None of this can live only in the current conversation. Agent sessions end, context compresses, and the next session starts without the old reasoning. We keep a small package under `goals/`:

| Artifact | What it holds | 
|---|---|
| `goal.md` | The objective, exclusions, and completion condition | 
| `facts.md` | Decisions that later rounds must not rediscover | 
| `plan.md` | The initial floor and expected capability order | 
| `LOOP.md` | The procedure, authority, budget, and stop rules | 
| `STATE.md` | The current queue, failures, and next action | 
| `corpus/` | Approved requests | 
| `rounds/` | Prompts, outputs, scores, screenshots, and decisions | 

`LOOP.md` changes when the process changes. `STATE.md` changes after each round. The distinction makes sessions disposable without making the work forgetful. A new agent reads the goal, procedure, state, and evidence, then continues from the same decision.

This is more than context preservation. The files make the development process inspectable. We can see why the loop chose a gap, which evidence supported the change, and whether the next run improved the same request.

## People still decide when it is enough

We run the inner loop in small batches. Three rounds usually produce enough evidence for a decision without giving the system room to drift. At the boundary, a person chooses whether to continue the queue, redirect the next batch, or stop. The loop stops sooner when it cannot reproduce or attribute a failure. It also stops when the world is unhealthy or the repair needs a frozen change.

This is the human-owned round from the convergence post. The agent owns the turns inside it. The boundary exists because the score cannot decide the product’s direction.

Most tickets do not need any of this. If complete tests describe the work, give the agent the tests and let it finish. The harness earns its cost when use must reveal the capability sequence. The system must also expose effects that the harness can observe.

Start with three requests, one fixture, one score command, one loop file, one state file, and a three-round budget. Add a new control only after a failure justifies it.

The quality of an agent loop is not how long it runs or how many commits it produces. It is what remains after a round. A good round leaves a working capability, evidence that it works, and a harness that will not pay for the same lesson twice. That is an agent loop worth leaving alone with a complex feature.
