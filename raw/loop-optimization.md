---
url: https://bradleygibbs.dev/posts/loop-optimization/
date_fetched: 2026-10-12
---

# Loop Engineering: Optimizing Code Generation and Verification

A practical map of AI-assisted development: qualify candidates for human review, reduce wasted work inside the build loop, and measure which checks and corrections earn their cost.

AI-assisted code generation can shorten the time needed to produce code while increasing the work required to understand and verify it. This article describes how to design processes that preserve human time, attention, and judgment: improve the process that develops candidates for evaluation, and organize the evidence humans need to assess them.

This is an overview of the four loops we’ll be optimizing.

- Catalogue of failure patterns, checks and responses
- Illustrative walkthrough with a contract, check results and human-review packet

## Introduction

Two fundamental challenges in AI-assisted software development are specifying intent precisely and completely, and verifying that the generated code satisfies that specification. This article focuses primarily on the second part of that equation: verifying the code that AI wrote.

AI-assisted code generation can produce much more code in the same amount of time. An NBER working paper covering more than 500,000 GitHub developers estimated that, in weeks 21–30 after adoption, interactive coding agents increased weekly lines changed by 958% and PRs created by 86%, while weekly repository releases increased by 20%. But writing and testing code is estimated to account for only about 25–35% of the time from initial idea to product launch (Bain). How does generating more code more quickly impact the rest of the software development lifecycle (SDLC)?

### The verification bottleneck that faster AI-assisted code generation can create

Faster, cheaper code generation can create new bottlenecks or worsen existing bottlenecks in downstream processes. Verification becomes more burdensome for five reasons:

- 
**More code arrives faster than humans can review it.**
- 
**Incorrect code still looks very plausible.**
- 
**Comprehension debt.**
- 
**Changes are bigger and carry more problems.**
- 
**The checks are unreliable too.**

We have been developing these methods for the past twelve months, and we have run our own pipeline and used it to generate code. To keep the example code clean and easy to follow, we chose illustrative examples, some of them hypothetical, over results from our own runs. This article does not report measured savings; the method is meant to be tested in your own workflow.

## 1. Terminal Evaluation

### What is terminal evaluation?

Terminal evaluation judges the outcome of an autonomous attempt and nothing else. Given a task and the criteria for success, an AI system works autonomously until it believes it has generated a result that satisfies the predefined criteria for success. Then an evaluator checks that result against the criteria. In a coding workflow, the result is a candidate and the criteria are the agreed specification. The verdict can be pass or fail, a partial-credit score or several scores. The final verdict tells us whether the outcome met the criteria, but nothing about how the system got there.

That verdict lets us ask two different questions:

- **Capability:**can this configured system perform the task? A successful result shows that it can, under the conditions tested.
- **Reliability:**how consistently does it succeed? Repeating the task under comparable conditions lets us estimate that consistency. A system that succeeds once has shown something different from one that succeeds in 99 of 100 attempts.

Both questions concern a configured system, not a model alone. The configuration includes the vendor’s model and harness (together, the vendor’s agent) and anything we add around it: a custom harness, tools, context, operating limits, a graph or a pipeline. Change any of these and the results can change, so capability and reliability must be remeasured for the new configuration.

For fair comparisons, use the same tasks and success criteria, keep budgets and retry policies comparable, and include unsuccessful attempts.

### Variations on a theme

Agent-based benchmarks (evals), prompt engineering, specification-driven development (SDD), and loop engineering are four variations on a shared underlying process: give AI a task, let it attempt the task autonomously, and then evaluate the result. Each variation emphasizes different parts of that process, but none prohibits adopting practices emphasized by the others.

For example, independent evaluators, enforced stopping conditions and automated handoffs are typically associated with loop engineering, but there is nothing preventing an SDD approach from adopting automated handoffs. Their presence depends on the system’s actual design. The methodology the developer believes they are implementing neither guarantees their inclusion nor prohibits their use.

*Figure 1: One process, four methods. Expand each approach to see practices and citations. The shared routes include rework, exhausted retries and escalation; none is exclusive to a method.*

### Writing the conformance contract cooperatively

In our coding workflow, humans approve the obligations before the run and decide whether to merge a qualified candidate afterward. These decisions place human judgment at both boundaries of the AI’s autonomous work. Humans and AI can cooperate in fulfilling both responsibilities. Osmani, Own the Outer Loop.

At the beginning of the process, humans and AI can work together to translate intentions into obligations: what the code must do, preserve and never permit. Rather than having humans write these obligations alone and then lob them over the wall at AI, humans and AI can work cooperatively to write and refine them and resolve any ambiguities as they arise. We call the approved obligations the * conformance contract*. Humans are ultimately accountable for ensuring that the conformance contract comprehensively expresses intent. The more accurately and completely intent is specified in the conformance contract, the harder it is for errors to slip by the evaluator.

### Convert obligations to checks

Each obligation also needs a way to assess conformance. Humans and AI develop the checks, expected results and evidence requirements before construction begins, identifying what can be checked automatically and what still needs human judgment. Tests, abstract syntax tree (AST) checks and runtime observations can supply evidence about particular properties and may suffice to show that some of the conformance contract’s obligations have been met by the candidate under consideration. Check design must address violations as well as intended behavior: a check that passes the intended implementation but also passes a defective one establishes little. The contract and its authoritative checks remain fixed during a bounded run. No AI agent may ever silently alter an obligation or a check during the course of a run. Instead, it may suggest changes, which, if accepted, would require re-running the process.

### Evaluating a candidate

When construction returns a candidate, controller code invokes an independent evaluator to assess it against the contract using those checks. The evaluator lives outside the build loop. Its checks, expected results and stopping rules sit outside the builder’s authority to change. It records which candidate was evaluated, which checks ran, what they found and what remains unverified. The builder’s completion message does not determine qualification, and missing evidence cannot count as a pass. Independence protects the assessment from changes made during construction; it does not eliminate gaps in requirements or check coverage.

The stopping conditions determine what happens next. A candidate that meets the qualification requirements advances to human review. A rejected candidate returns to construction with the failed obligations and supporting findings, provided retries remain. Exhausted retries stop the run without a qualified candidate. An ambiguity or other issue requiring human judgment pauses the process and raises the question. These decisions are enforced by the controller, rather than left to the builder’s willingness to continue or declare itself done.

### Human review

Once the evaluator qualifies a candidate, the human must decide whether to accept it. This is where the checks prepared before the run pay off. What can be verified deterministically should be: tests, AST checks and runtime observations settle the obligations they can, and their recorded results are established facts the reviewer does not need to re-derive. What remains are the claims no mechanical check can settle, such as whether the requirements captured the intent, whether the behavior is what the product needs, and how the change fits the surrounding system. AI assembles a view for each obligation that brings together the obligation itself, the checks that exercise it and their results, the code generated to meet it, the unresolved questions and any other evidence related to it, so the reviewer starts from the claims that need judgment instead of reconstructing the evidence from scratch.

In this way, deterministic verification directs the human’s budget of attention and judgment to the claims that need it. Dependable qualification lets reviewers rely on established results and focus on unresolved judgments, spending their effort where the evidence runs out. That reliance depends on check coverage and evidence tied to the candidate, with uncertainty as visible as passing results; a weak check, or one that passes a defective implementation, establishes little. Reviewers still need to understand how the change fits the surrounding system and what its consequences are. AI can help assemble and examine the evidence even where it cannot settle that judgment. The human decides whether to merge.

### An example: a storage-read failure

Consider an application that cannot read previously saved data. Its contract requires it to report the failure and preserve the original data. A candidate that reports the error but overwrites the data with an empty list satisfies only half of that obligation. The evaluator must check both the failure response and preservation of the stored data.

An accompanying walkthrough works through this example, with sample documents showing the conformance contract, executable checks, rejection of the defective candidate, repair and handoff for human review. Its review packet connects the requirement to the changed code and check results, while identifying questions the reviewer still needs to resolve. Does the application use the evaluated code path? Are the selected cases sufficient? Is the recovery behavior what the product needs? The candidate history is illustrative, not a recorded run.

### What terminal outcomes leave unexplained

When we measure only terminal outcomes, we treat the autonomous process of developing a candidate as a black box. The verdict does not show the intermediate plans, changes, checks or corrections. One successful attempt might proceed without error; another might introduce several errors and recover from all of them. Both receive the same success verdict. A failed attempt tells us that the system did not meet the criteria, but the verdict alone does not explain where or why it failed. Terminal verdicts have **low diagnostic resolution**: they report the outcome while leaving much of the process unexplained.

Understanding those differences requires examining the intermediate work and how one result shaped the next. We need to see where errors arose, whether they propagated or were corrected, and what those corrections cost in time and tokens. Terminal verdicts alone cannot answer those questions. Broader reliability evaluations also examine trajectories, robustness and the consequences of failure. Princeton reliability study.

### Thought experiment: Assuming evaluation is perfect

For the sake of argument, assume the conformance contract captures all intent and the evaluator and qualification rules are perfect. Only code conforming to that fully captured intent can be qualified and presented for human review. This is a thought experiment, not a claim about current systems. It guarantees the correctness of a qualified candidate, not that construction will produce one within budget.

Even with that perfect gate, the build process can make mistakes, recover and waste work on the way to producing a correct candidate. What could earlier detection and correction save in time, tokens and human attention? Could earlier detection and correction mean that a qualified candidate is produced within the given retry budget? Those are the questions we’ll examine next as we take a look inside the build loop. Possible reductions in false greens (candidates that pass despite violating an obligation) belong to the later discussion, when we relax the perfect-evaluation assumption.

## 2. Engineering the Inner Loop

Return to the storage-read example from Section 1. The walkthrough’s conformance contract includes obligation TD-08: when stored data is invalid, unreadable or unsupported, the application must return a distinguishable read-failure result and must not overwrite or clear the original stored bytes. We will follow the preservation half of that obligation through preparation, a planning handoff and the checks that could catch its omission.

Within the build loop, an agent, pipeline or graph plans, writes, checks and revises in order to construct code that meets the specifications it has been given. Build and external evaluation form the inner loop; human design and review bound the outer loop. The diagram below opens up the build loop: preparation, action, optional checks and corrections, and submission to the evaluator.

Under our perfect-evaluator assumption, external evaluation inside the inner loop is what establishes the correctness of generated code candidates. Let’s look into the inner workings of the build loop to see how we can improve the path to producing candidates for the evaluator.

We focus on two sources of errors in the build loop, which call for different responses. They are not the only possible sources; capability limits, tool faults and defects in the contract or checks also matter:

- **Environmental (missing prerequisites):**
- **Agent failure modes:**

One study of coding-agent failures suggests why the second source matters. Failure as a Process examines how terminal-based coding agents fail. The researchers ran seven frontier models in each of three coding-agent scaffolds on Terminal-Bench tasks and kept 1,184 failed and 610 successful trajectories. For each failed run, they identified a single decisive error, the one that determines the eventual failure, identified in hindsight and not necessarily the first to appear, and classified its root cause. Most decisive errors (57.9%) were epistemic rather than competence-related: the agent mishandled information it already had. Their categories include specification neglect, output misreading and ignored signals. They also found that successful runs contained errors followed by recovery. Together, these findings support examining the process behind a verdict, not only the verdict.

The study is useful for our purposes, but it answers a different question than ours. Its categories overlap with what we call agent failure modes (defined in Section 2, “Distinguish missing prerequisites from agent mistakes”), though the overlap is not identity. They are not defined by our narrower test of whether the agent had sufficient, accurate and usable information at the relevant step, so we cannot say how many of the errors it counts would qualify under our definition. The study also counts one decisive error per failed run, while in our own runs we have often seen several failure modes within a single run, so its shares cannot tell us how often each pattern occurs overall. And although it analyzes recovery, it does not separate out errors of our kind, so it does not tell us how many of them can be recovered, whether by the agent unaided or with an intervention.

Learning what we want to know would take further work: annotating every error in a run, not only the decisive one; classifying each against our definition, which means establishing what information the agent had at that step; measuring how often each class of error is recovered, unaided and with an intervention such as a targeted check or correction; and repeating those measurements across models, harnesses and tasks, with enough repeated runs to account for variation between attempts. Until then, treat 57.9% as evidence that mishandling available information is common and worth examining, not as a rate we can apply to our own loop. We leave that work to follow-up studies.

The companion walkthrough, Following a Candidate from Conformance Contract to Human Review, follows one requirement from conformance contract to human review: obligation TD-08, described at the start of this section. Here we look inside the build loop at how errors can arise on the way to a candidate that meets it. In this example, a planning stage that is never told that a failed read must preserve the stored data has an environmental problem. A planning stage that is told, and still plans to replace unreadable data with an empty store, has a failure mode.

The two sources arise in different places in the build loop, so we handle them differently:

| Source of error | In the storage-read example | Where it arises | Main response | 
|---|---|---|---|
| Environmental:the agent lacks what it needs | The planning stage is never given obligation TD-08, or does not know that stored data can be unsupported or unreadable. | In how a stage is set up: the obligations, context, tools and handoffs it receives. | Prevent it by setting the stage for success. Detect it by inspecting what actually reached the stage. | 
| Agent failure mode:the agent has what it needs and still errs | The planning stage has the obligation and the relevant facts, but its plan replaces unreadable data with an empty store instead of preserving it. | In the agent’s own decisions and actions at a stage, often at a handoff or when it reports a result. | Reduce exposure through how the work is arranged, then observe what happens, check at the right point and correct. | 

Many real incidents mix both, and the available records often cannot say which occurred. We treat that uncertainty as information to preserve, not a nuisance to resolve by guessing. The three activities below follow the table: the first mainly prevents environmental errors. Observation and correction apply to either source: a check can establish a defect, and justify repair, before its cause is known.

The labels at the foot of the steps in the diagram above mark where each of the three activities described next applies.

Three activities improve the path to producing candidates for the evaluator:

- **Set the stage(s) for success**(the PREPARE step). Prepare the context and tools each stage needs, and arrange the work to avoid preventable errors.
- **Observe what happens**(the CHECK step). Check intermediate work for observable errors, separate missing prerequisites from agent mistakes, and record what occurs before changing anything.
- **Intervene when the evidence warrants it**(the DECIDE and CORRECT + VERIFY steps). Decide whether the evidence warrants a correction, then correct the work and verify the correction, at the earliest point where the evidence is reliable.

The ACT step is the agent’s own work, and SUBMIT hands the candidate to the evaluator. The three activities improve what happens around them.

A companion guide, Engineering the Inner Loop: Preparation, Failure Modes and Responses, supports all three, and we point to it as we go. It starts from the cause of a difficulty, because an incomplete specification, missing information or a failing tool needs a different remedy than a mistake by the agent. It then pairs common problems, such as unclear objectives, stale information, too much context, ambiguous authority, oversized tasks and weak evaluation, with a design practice, the evidence that shows it worked and the cost to measure. Next, it catalogues fifteen recurring agent behaviors, for example an unfaithful handoff, skipped work and unsupported claims that checks passed, each with a proposed check and response. Finally, it shows how the learning loop chooses which mechanisms to keep. Use it as a design checklist whenever you define an agent’s task, a pipeline stage or the edge between two stages. Its patterns and checks are working guidance to test, not an exhaustive taxonomy or a validated detection suite.

### Activity 1: Set the stage(s) for success (PREPARE)

A **stage** is one unit of work inside the build loop: a step in a single agent’s session, a role in a pipeline such as planning, implementation or review, or a node in a graph. Stages pass work to one another through **handoffs**. How well each stage is set up, and how the stages and handoffs are arranged, decides how many errors reach the later checks at all. The guidance below comes from our General Principles for working with agents, which cover supportive environments, context strategy, verification and implementation. The companion’s section on preparing the work collects it.

#### Give each stage what it needs

Give each stage a clear responsibility, the relevant obligations, accurate application and API knowledge, suitable tools, and access to further information. Preserve the requirements and unresolved questions when work moves between stages. Enforce mechanically checkable permissions and mandatory transitions in controller code. The companion’s preventive design guidance brings these practices together.

Supplying everything at once is not the same as supplying useful context. Anthropic’s context-engineering guidance describes progressive retrieval and the risk of losing important details during compaction. Narrower tasks and specialist agents can help manage information, but every handoff introduces another opportunity to lose it. For the storage-read example, both parts of TD-08, reporting the failure and preserving the stored data, along with the relevant facts about the storage interface, need to reach the stages making decisions about them.

#### Design stages and handoffs deliberately

Before adding a detector, ask whether the arrangement of work is creating the difficulty. In a planning–implementation–review pipeline, the implementation stage needs the controlling obligations and relevant application facts, not just the planner’s conclusions. A handoff should carry references to those sources, the decisions already made and the questions still open. It should identify which candidate or starting state it describes. These requirements apply equally to stages within one agent’s session and to separate agents.

A useful way to design each stage is to answer five questions: what is it responsible for, what must it know, what may it change, what must it return, and what happens if it cannot finish? Then examine the edges between stages. If two workers can edit the same state, how are their changes reconciled? If a worker returns an incomplete result, can later work distinguish that from completion? Where the answer is mechanically enforceable, put the rule in controller code. Where it requires interpretation, preserve the evidence and an escalation route.

For our planning stage, that means planning how the application responds when stored data cannot be read, knowing how storage is loaded and saved, and having authority to propose implementation work while leaving the approved obligations fixed. Its handoff must preserve both parts of the obligation, reporting the failure and leaving the stored bytes untouched, and identify unresolved questions. Missing facts about the storage interface should lead to retrieval or an explicit block before implementation relies on the plan.

Splitting work can reduce the information each agent must handle, but it can also separate facts that need to be considered together. Keeping failed-read reporting and stored-data preservation in one assignment may be preferable to delegating them independently and repairing integration errors afterward. A separate reviewer might add useful scrutiny or simply repeat the same misunderstanding. Treat topology as a design choice to test. Neither a longer prompt nor a larger graph is inherently the solution.

### Activity 2: Observe what happens (CHECK)

Before changing the loop, we need to know what is actually going wrong. Separating the two error sources helps distinguish problems in requirements, information delivery, tools and coordination from errors in how an agent uses what it received.

#### Distinguish missing prerequisites from agent mistakes

For this article, an **agent failure mode** is a recurring pattern of incorrect decisions or actions on tasks within the configured agent’s demonstrated capabilities, despite having sufficient, accurate and usable knowledge and context at the relevant step. A **failure-mode-induced error** is an observed instance supported as matching that pattern. Examples include neglecting an explicit requirement, misreading clear tool output or reporting a check as passed despite a recorded failure.

The distinction matters. An agent missing essential API knowledge needs that knowledge supplied. An agent that receives and neglects an explicit requirement presents a different problem. A confirmed defect can warrant repair while its cause remains unresolved. These patterns need not appear on every attempt, and neither spontaneous recovery nor an instructed correction is guaranteed.

The inner-loop engineering companion combines the General Principles with a catalogue of agent behavior. Missing knowledge, excessive context and incomplete specifications belong in its preventive design guidance; the behavioral entries address mistakes despite adequate prerequisites. Each entry connects a recognizable pattern to a proposed check, a response and verification of the correction. Use these to choose mechanisms to investigate; they are not an exhaustive taxonomy or a validated detection suite.

#### Observe before intervening

One practical starting point is to run a proposed detector in observation mode: record its findings without altering construction, then examine what happened next. For the storage-read example, record whether the planning handoff preserves the stored data on a failed read and what the implementation does with it. Did the agent recover? Did the evaluator reject the candidate? Did the signal concern a real defect? This can reveal poor placement or excessive false alarms before adding interruptions. It does not tell us what would have happened with a correction; that needs the comparative runs discussed below.

### Activity 3: Intervene when the evidence warrants it (DECIDE & CORRECT + VERIFY)

#### Follow one omission through the loop

Return to the storage-read example. The planning agent receives obligation TD-08 and the relevant facts about the storage interface; assume it has demonstrated the ability to handle this kind of change.

In this hypothetical, its handoff nevertheless plans to recover from an unsupported stored version by saving an empty store and then reporting the read failure. The plan reports the failure but drops the requirement to preserve the original data. An implementation stage follows that plan, and the stored data is overwritten. The source contract was never changed. The error arose when an intermediate artifact lost an obligation and later work relied on it. This illustrates the companion’s unfaithful handoff pattern.

The implementer could reread the contract and recover. Under our perfect-evaluator assumption, the final candidate would be rejected if it did not. Neither possibility makes the omission harmless to the process: dependent code and tests may have been built around it before correction begins. A downstream agent receiving only the shortened handoff also has an information gap; that does not automatically constitute another independent failure-mode activation.

Where could we intervene? At the handoff, compare the proposed work with the relevant source obligations before dependent implementation begins. An explicit list of obligation identifiers allows a mechanical check for missing references. Confirming that the plan actually preserves their meaning may require semantic review. A complete list alone is insufficient: in our example, a plan that cites TD-08 by identifier but proposes overwriting the data would pass the first check and fail the second.

If the omission is confirmed, return a specific finding: the plan replaces unreadable data with an empty store, contradicting TD-08’s requirement to preserve the original bytes; identify the controlling obligation and the affected work. Restore it to the plan and repair any implementation or tests already derived from the omission. Do not weaken the contract to fit the candidate.

Then verify the correction behaviorally, using the contract’s four representative failures: an unsupported version, a stored value that is not an object, a version-1 envelope whose `todos` value is not an array, and an adapter read error. For each, the application should return a read-failure result, leave the stored bytes unchanged and make no save calls. Absent data should still yield an empty list; this checks that the repair does not simply turn every unusual read into an error. Also run the checks against a deliberately defective version that overwrites the data on a failed read, and confirm that it fails because the stored bytes changed. That shows the checks can detect this particular defect. These checks exercise four concrete cases, not every possible malformed input.

The useful evidence is the source obligation, incomplete handoff, dependent change, finding, repair and executed check results. Keeping that chain lets a reviewer inspect what was corrected without reconstructing the entire attempt. It also lets us test whether the intervention helped.

#### A check needs a target and a response

The storage-read example gives a failure-mode label an operational meaning: an observable omission, a checkpoint before propagation, a bounded repair and a way to verify it. Other patterns require different mechanisms. A claim that tests passed can be compared with execution records. Repeated unsuccessful repairs can be investigated through candidate changes and repeated findings. Neither check requires guessing the model’s internal reasoning.

A generic instruction to “look for failure modes” leaves those decisions unresolved. Choose where the relevant evidence becomes available, what constitutes a finding and whether the response should be correction, further investigation, continued work or escalation. A mechanical check can compare identifiers or execution records. A model-assisted check can assess whether a summary preserves a requirement’s meaning, but its judgment needs qualification. Anthropic’s agent-evaluation guide discusses these grader tradeoffs, including calibration of model judgments against human assessments.

For the handoff detector, prepare several small cases: a plan that preserves the stored data on a failed read; one that omits the preservation requirement; one that mentions TD-08 by identifier but proposes overwriting the data; and one that expresses preservation correctly in different words. The check should distinguish the omission and contradiction without rejecting a legitimate alternative phrasing, and without flagging the legitimate creation of an empty store when no data has been saved yet. Include cases it was not tuned on. This tests whether the mechanism detects the intended defect, rather than whether it recognizes the vocabulary in its own examples.

Test the correction separately. A detector can identify a missing obligation correctly and still prompt an ineffective repair. The agent might add a sentence to the plan while leaving the implementation unchanged, or stop overwriting the data but start treating absent storage as an error, contradicting the requirement that absent data yields an empty list. Supply the specific finding and relevant source, bound the permitted repair, and verify the affected behavior and preservation obligations afterward. Repeated failed repairs need a limit and an honest stop or escalation; adding a correction agent does not make correction dependable by definition.

Timing is another choice. The useful checkpoint for a handoff is when it is ready to guide dependent work. An unfinished plan may legitimately omit details that the author is still adding. Likewise, a transient test failure during an incomplete edit may not justify interrupting the agent. Intervene at the earliest point where the evidence reliably establishes a problem and the avoided downstream work is worth the interruption. Some constraints must be enforced before an action; optional quality checks can often wait for a coherent artifact.

Early detection remains a substantive problem. In *Failure as a Process*, an experimental monitor read partial trajectories and tried to identify runs that had passed a retrospectively labeled “lock-in” point, after which no successful recovery was observed. Its best recall was 28.8%. This was a different task from detecting a specific omission in a completed handoff, but it illustrates the difficulty of identifying trouble from partial histories. Monitor results. A catalogue gives us targets for investigation, not a detector merely by naming them.

As the build-loop diagram above shows, these optional checks and responses sit alongside ordinary recovery and submission to external evaluation.

### What might the intervention buy us?

Compare three possible histories: the agent notices the omission itself; the external evaluator rejects the candidate and prompts repair; or an intermediate check catches it before more work depends on it. Early intervention may save rework. It may also interrupt cheap self-recovery, consume tokens on a false alarm or introduce a new defect. Its value depends on the difference between these histories, including the cost of checking and verifying repairs.

There is evidence that targeted intervention can help. Meta’s Wink periodically examined coding-agent histories and supplied corrective guidance. Its 15-day production A/B test reported 5.3% fewer tokens and 4.2% fewer engineer interventions per session. Those are operational improvements in that setting; they do not establish final code correctness or equivalent savings in another loop. The paper’s separate recovery assessment judged cessation of the detected behavior and renewed progress, rather than independently verified task completion.

Suppose, for a second thought experiment, we could prevent or correct every instance of the behavioral patterns we target, including their downstream effects. Candidates would be free of defects caused by those instances. Eliminating that error class would not by itself establish complete requirements, sufficient knowledge, correct tools or the capability to finish the task. We would still need to measure the work saved: some errors would otherwise have been corrected cheaply, while others might have driven extensive rework. The empirical question concerns changed outcomes and costs, not simply how much of a taxonomy a detector covers.

* With perfect terminal evaluation, early checks add no correctness guarantee to qualified code.* When we relax that assumption, an additional check might expose a violation the evaluator misses and prevent a false green. That is a potential additional benefit. The primary motivation for inner-loop engineering is to reduce the time and tokens (and potential rework or retries) required to reach a conforming candidate.

Which checks earn their cost, for our tasks and agent configuration? That is the learning loop’s job.

## 3. The Learning Loop

Improving preparation or adding a check can change the work required to produce a conforming candidate. Whether that change helps depends on what would otherwise have happened, including ordinary recovery, evaluator-driven rework and the intervention’s own costs.

- Gather evidence from design, construction, evaluation, human review and defects found after release.
- Turn an observed problem into a testable change, then compare detection, repair, overall outcomes and human effort.
- Reassess support when the agent or its working conditions change, and recommend which mechanisms to retain, revise or retire.

The learning loop uses evidence from both successful and failed runs to recommend improvements to the process. Lilian Weng’s harness-engineering survey examines the broader idea of making the harness itself an optimization target. In the arrangement proposed here, AI analyzes and recommends; humans approve changes to the loop.

### What the learning loop receives and recommends

#### Sources and the evidence they supply

The learning loop needs evidence from every part of the process, not only from construction. A bug reported two weeks after release may expose an omitted obligation, a weak check or a failed repair, and each points to a different improvement. Its inputs come from five sources:

| Source | Evidence it supplies | 
|---|---|
| Human design | The versioned contract, its rationale, assumptions and unresolved questions, and the mapping from obligations to checks. | 
| Build loop | What each stage received and returned, check results, interventions and repairs, failed attempts and recoveries. Also the agent and harness configuration, context and tool versions, run limits, time and token usage, including unsuccessful runs. | 
| External evaluation | Per-obligation findings, rejected candidates, the stopping rule applied and why, retries used and evaluation cost. Unavailable evidence stays visible. | 
| Human review | The candidate and evidence package reviewed, the questions raised, review time and effort, and the decision with its rationale and requested changes. | 
| After release | Defects, regressions and incidents found after merge, traced back to the change that introduced them. | 

#### How the evidence becomes a recommendation

The learning loop then works in four steps:

- **Reconstruct what happened.**Trace each problem to the earliest evidenced error, the conditions that contributed and the checks that missed it. Separating the two error sources helps diagnose what went wrong, separating missing information, an incomplete specification, a weak evaluator, tool or coordination failures and agent failure modes. Where the records cannot say, the cause stays unknown.
- **Match the problem to candidate changes.**The General Principles suggest how to prepare the work. The behavioral catalogue suggests checks and responses.
- **Propose a bounded change.**State the hypothesis and what evidence would count against it.
- **Test it and recommend.**Compare outcomes and costs, estimate detection and correction rates, and recommend keeping, narrowing, revising or retiring the change. A human approves it; requalification revisits the decision as conditions change.

#### Where recommendations land

Because the evidence spans the whole process, a recommendation can point to any part of it:

- **Contract preparation and review.**If the contract never required preserving stored data on a failed read, improving the handoff checker cannot recover that missing intent.
- **Context preparation and stage design.**Better API context, a simpler workflow or a different split of work may remove an information gap that a detector would otherwise catch late.
- **Detectors and corrections.**Add, narrow, revise or retire a check or its response.
- **Evaluation.**If the obligation was present but the test exercised the wrong path, improve the check.
- **Evidence handoff.**If a reviewer spent an hour locating an already recorded result, improve how the evidence is assembled.

### Turn an observation into a testable change

Use the illustrative storage-read history to formulate a bounded proposal: compare the planning handoff with its source obligations before implementation, because repairing a plan may cost less than repairing work derived from it. Specify the checkpoint, the finding it can return and the permitted correction. State what would count against the proposal: frequent false alarms, no improvement over ordinary recovery, or repairs that introduce other defects. This turns an omission like this one into a hypothesis we can test.

### Compare the whole process

Test the proposed handoff check on repeated comparable tasks with and without the intervention, keeping the contract, evaluator, agent configuration and budget policy fixed. The baseline retains ordinary agent recovery and evaluator-driven rework. Compare both paths through qualification or an unsuccessful stop, including checking and repair overhead. Include representative cases not used to develop the check; one successful correction cannot establish dependable benefit.

Keep four questions separate: did an error occur, did the detector identify it, did the correction work, and did the overall run improve? Review sampled histories independently of the detector to look for missed errors and false alarms. Preserve uncertainty about causes. Several labels attached to one omission do not create several incidents.

Start each comparison from an equivalent clean state, without passing one attempt’s repair into the next. Where practical, randomize or interleave the conditions to reduce the chance that changing service conditions explain the result. A small set of handcrafted faulty handoffs can test detection; it cannot tell us how often those handoffs arise in ordinary work. Use representative tasks to estimate practical value, and report how many tasks and repetitions support the result.

The accounting should include:

- **Detection:**confirmed incidents per relevant opportunity, missed errors, false alarms and how early the check caught a problem.
- **Total cost:**preparation, coordination, every check invocation, correction and rechecking, including time and available token usage from unsuccessful runs. Include setup and maintenance effort.
- **Outcomes and consequences:**completion within budget, regressions, escaped defects and the potential damage if an error goes undetected. A rare data-loss defect may justify a check that frequency alone would dismiss.
- **Human effort:**supervision, clarification, reviewing evidence and reconstructing missing context. Less involvement during construction can mean less familiarity at review; count both.

A frequently triggered check may interrupt inexpensive self-recovery. A quieter check may miss errors or prevent a consequential escape. Estimate the likelihood and consequences of an escape where evidence permits, and state uncertainty where it does not. Signal counts alone cannot decide which mechanism belongs in the loop.

For a simple arithmetic illustration, a check costing 20 seconds on each of 100 invocations consumes 33 minutes and 20 seconds of cumulative checking time before any repairs. If it finds ten real problems, the time saved across those cases must cover that total overhead as well as correction and rechecking for the mechanism to save execution time overall. Those ten detections are not ten rescued runs. Parallel execution also means cumulative work and wall-clock latency are different measurements. Keep time, tokens and human effort distinct rather than collapsing them into a single productivity score.

The decision can still favor a mechanism that adds execution cost if it materially reduces consequential escapes or human work. Conversely, a faster run that hands reviewers a less intelligible candidate may have moved the burden rather than reduced it. Measure the evidence package itself: can a reviewer find the controlling requirement, inspect the relevant change and see both supporting results and unresolved limits? Can they explain the changed behavior and recognize a consequential gap in its evidence? On comparable review cases, assess those judgments alongside time spent, including the effort required to correct misleading evidence. A shorter review is useful when it preserves or improves the quality of the decision.

The same comparison applies to preventive design. Better API context might eliminate an information gap more effectively than adding another reviewer. A simpler workflow might avoid handoff losses. Test one change at a time where practical, or identify the combination being tested. Preserve the full history so a final green result does not hide repeated failures or an expensive route to completion.

### Treat detection and correction as rates

Agent behavior can be stochastic: the same task and working conditions can lead to different errors or different responses to a correction. One successful repair shows that the correction can work, not that it will work each time the error arises. A particular trigger can still produce a highly repeatable error. Record the conditions under which each pattern appears and is corrected, so the rates describe the situations in which the mechanism will be used.

In Wink’s evaluation, an LLM judge rated agents as recovered in about 91% of single-intervention conversations and about 79% of conversations with multiple interventions. Recovery meant that the detected behavior ceased and the agent made forward progress in the subsequent steps. These are judged recovery rates, not guarantees or independently verified task-completion rates.

Treat each link in the chain as a rate to estimate: how often the error type arises across opportunities, how often the detector catches it, and how often the correction succeeds once it has. Estimate each from repeated comparable cases and report how many support it; a handful of observations leaves wide uncertainty in both directions. Three successes in three observed cases is compatible with a success rate well below 100%.

Those rates should inform the decision to add or remove a detection or correction mechanism, alongside its cost and the consequence of an escape. A correction that works only some of the time can still earn its place when the error is frequent or consequential. Check each attempted repair before dependent work relies on it; a failed or unverified repair needs a bounded retry, an honest stop or escalation. The repair check has its own coverage limits, and a correction never qualifies the candidate by itself. A mechanism that is rarely needed, or rarely works, may not justify its overhead. Keep measuring as the model, harness and task mix change.

### Requalify as the system changes

A new model or vendor harness can change which support is useful. Anthropic reports removing context resets when Opus 4.5 largely resolved the premature-wrapping-up behavior that had motivated them with Sonnet 4.5; automatic compaction still handled context growth. That is a concrete example of revising support for a particular configuration. Harness design report.

Repeat the comparison after material changes to the model, harness, tools, context strategy or task mix. Include prior failures and representative new cases, and test the optional mechanism both enabled and disabled. Recheck the detector, too: fewer alerts could mean fewer errors, fewer opportunities or a changed error pattern it no longer recognizes. If evaluation changes, assess that change separately before interpreting a higher pass rate.

When both agent versions are available, four conditions separate improvement in the agent from improvement supplied by the mechanism:

| Configuration | Optional check disabled | Optional check and correction enabled | 
|---|---|---|
| Earlier agent | Establish its ordinary recovery and evaluator-driven rework. | Estimate the mechanism’s contribution with that agent. | 
| Updated agent | Measure the updated agent without the custom support. | Determine whether the support still adds enough value. | 

Use the same task set and acceptance criteria across these conditions, with repeated attempts and recorded costs. A lower error rate with the check enabled cannot establish that the agent has taken over its responsibility. The disabled condition is essential to that decision. If an older configuration cannot be reproduced, state that limitation and qualify the current arrangement on its own evidence.

This gives “the model eating the harness” a practical test. Can the vendor agent perform this responsibility dependably enough under our conditions, with acceptable costs and consequences? Evidence can justify retaining, narrowing, improving or retiring custom support. The responsibility remains when its implementation moves. Removing an early intervention does not itself justify removing independent qualification or the human’s acceptance decision.

There is no universal threshold for keeping a check. A team may retain it only for security-sensitive changes, run it at handoffs instead of after every action, or replace a semantic review with a cheaper mechanical check for a narrower property. Record the scope of that decision and the evidence supporting it. Continue observing the relevant outcomes after a change so the learning loop can recommend restoring support if regressions emerge.

The useful output is a recommendation tied to evidence: what changed, which outcomes improved or worsened, what it cost and what remains uncertain. Human approval closes the learning loop. Start with one recurring problem or consequential obligation, test a focused improvement, and keep checking two outcomes: the time and tokens required to reach a conforming candidate, and the usefulness of its evidence for human review. If you have tried this, which mechanisms earned their place—and which turned out to cost more than they saved?
