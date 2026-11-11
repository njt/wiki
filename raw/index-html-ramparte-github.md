---
url: https://ramparte.github.io/agent-building-playbook/index.html
date_fetched: 2026-11-11
---

Patterns for building better in the agentic age. Generated from the curated patterns/ source — each pattern is listed once; filter by dimension to narrow the list.

"Four Moves: Write, Select, Compress, Isolate"Every context management decision is one of four moves — write it out, select only the relevant part, compress the verbose into the essential, or isolate the detail into a sub-agent.context-engineering

"Long-Horizon Memory: Compaction, Notes, Isolation"Long-running tasks require three memory strategies — compacting history as it ages, maintaining running notes files, and isolating sub-tasks to separate sessions to prevent context collapse.context-engineering

"Thin Pointers, Zero Poisoning"Pass file paths and references between agents instead of full content — let each agent pull only what it needs, only when it needs it.context-engineeringorchestration

Ablate the Harness as Models ImprovePeriodically have a stronger model inspect your workflow and delete steps that are no longer needed, shorten prompts, and merge nodes — because model capability changes faster than workflow habits, and yesterday's scaffolding becomes today's tax.workflow-disciplinemeta-principlescost-routing

Add Autonomy Only When JustifiedEvery degree of autonomy you give an agent must be paid for with oversight proportional to the risk — add it only when the cost-benefit calculation is explicit and positive.orchestration

Adversarial Parallel ReviewRun several reviewers with different models or roles and little shared context, reward significant findings over volume, then have a meta-reviewer rank what matters — because an implementer shares the blind spots that produced its own bugs.verificationreliabilityorchestration

Agents Amplify ExperienceAgents multiply the output of whoever wields them — great judgment gets amplified into great results; poor judgment gets amplified into confident mistakes at scale.taste

Anchor on Canonical NamesA single well-known domain term — "TDD, London School", "arc42" — activates a whole coherent methodology the model already learned in training, far more reliably than a paragraph paraphrasing the same idea, so name the concept instead of describing it.context-engineeringmeta-principles

Attention Is the Scarce ResourceWhen agents generate more work, options, and output than humans can inspect, the bottleneck is no longer machine throughput but human attention — so treat attention as a managed, budgeted resource, not an infinite input.human-factorsmeta-principlesobservability

Automate the Easy, Avoid the Mystery HouseAutomate what you understand completely; never automate your way into a system where nobody knows why it works or what it does.tasteworkflow-discipline

Bite-Sized TasksBreak work into units small enough that each completes, commits, and can be reviewed independently.workflow-discipline

Brownfield Is the Real TestGreenfield prototypes are easy to celebrate; brownfield systems expose whether a workflow can handle legacy constraints, hidden dependencies, and production risk — the mature move is to characterize existing behavior and write regression tests before any rewrite, never a blind rewrite.workflow-disciplinereliabilityverification

Build a Pattern RegistryTeams independently rediscover the same loops, validators, and review tricks — a pattern registry captures the reusable operating moves with usage guidance and contraindications, so practice transfers without forcing everyone onto the same stack; it is the package manager of agentic practice.knowledgemeta-principlesworkflow-discipline

Build an Awareness Layer That Watches Your WorkA separate process or agent that monitors the primary agent's actions in real time catches drift, failures, and unexpected behavior before they compound — the watcher that the working agent cannot be for itself.observability

Build Five to Throw AwayWhen you don't understand the requirements yet, build several variants in parallel — comparing them reveals what you actually care about.tasteworkflow-discipline

Build From Three PrimitivesEvery agentic system is built from three primitives — LLM call, tool call, and loop. Understand these primitives completely before reaching for higher-level abstractions.orchestrationtool-design

Checkpoint to a Handoff File, Then Restart CleanWhen context grows unwieldy or a session must end, write a structured handoff file capturing exactly what the next session needs, then start fresh with only that file.context-engineeringworkflow-discipline

Close the Loop From Production Back to FixFinishing doesn't end at deployment — route support messages, incidents, metrics, and user reports into a triage loop that proposes a fix, validates it, ships it behind a flag, and watches whether the signal disappears; this is the agentic version of a Toyota production system.orchestrationobservabilityworkflow-discipline

Coaches and PlayersLeverage no longer comes only from senior people doing senior work directly — it comes from injecting senior judgment into many agent-driven workflows; coaches frame problems and steer, players drive the agents day to day.human-factorsorchestrationmeta-principles

Code Execution Beats Loading Hundreds of Tool DefsA single code-execution tool — bash, Python, a REPL — often replaces dozens of specialized tools and avoids the context cost of registering a large tool set that the agent must reason over on every call.tool-designcontext-engineering

Code Is Cheap, Maintenance Isn'tAI-generated code is free as in puppies — someone still has to feed it, secure it, and maintain it for as long as it lives.meta-principles

Composable Orchestration PatternsBuild orchestration from a small set of named, reusable primitives — sequences, conditionals, fan-outs, and loops — rather than writing bespoke control flow for every workflow.orchestration

Context Engineering Over Prompt EngineeringThe model's output quality is bounded by context quality — sculpting what the model sees matters more than sculpting the instruction text.context-engineering

Context Is a Finite ResourceCurate the smallest high-signal token set — every token of noise degrades every token of signal.meta-principlescontext-engineering

Critic ApplicationsBuild a deterministic tool — a binary, parser, validator, or CLI — that acts as law from the agent's perspective; it cannot be negotiated with, only satisfied, which compresses complex intent into a checkable affordance and compensates for model weakness.tool-designverificationreliability

Decision TheaterBuild a small interactive artifact — a model, simulator, spreadsheet, or scenario explorer — that lets stakeholders change assumptions instead of debating abstractions; if they disagree with the conclusion, they must reveal which assumption they reject.intentworkflow-disciplinehuman-factors

Decompose and Retry on TimeoutWhen a task times out, the answer is almost never to raise the timeout — it is to break the task into smaller pieces.workflow-disciplineorchestration

Default to a Single-Threaded Linear AgentRun one task at a time in sequence unless you have a concrete reason to fan out — parallelism is an optimization, not a starting point.orchestration

Default to Clean-Slate DelegationWhen spawning a subagent, start with an empty context and provide only what the task needs — shared history is a source of inherited confusion, not free context.orchestrationcontext-engineering

Delegation Is Token EconomicsEvery delegation decision is a token investment — the tokens spent spawning and briefing a subagent must be justified by the value of offloading the work from the orchestrator's context budget.cost-routingcontext-engineeringorchestration

Demand Independent Proof, Not a Status ReportAn agent's claim of success is a hypothesis — treat it as one until you hold evidence that no longer originates from the agent itself.reliability

Design for Emotional OwnershipA system that technically works can still fail to be adopted — fully headless automation can feel alienating, and people who feel no ownership of an output won't trust it; design the human interface to autonomy with visible progress, proof artifacts, and moments of authorship.human-factorsobservabilitytaste

Design Tools for Agents, Not HumansAgent tools should be structured for machine parsing — unambiguous outputs, complete information in every response, no UI affordances that only make sense for humans.tool-design

Designing the Agentic Loop Is the Core SkillThe quality of an agentic system is determined almost entirely by the design of its loop — how the agent perceives, decides, acts, and verifies before cycling again.verification

Develop Your TasteTaste — the ability to recognize quality and know when something is wrong — is a craft that must be deliberately built; it does not emerge from volume of output alone.taste

Distinguish Workflows From AgentsA workflow is a fixed sequence of steps with known branching; an agent decides dynamically what to do next. Conflating the two produces systems that are harder to test, monitor, and trust.orchestration

Document Intent, Not Just MethodCode describes how; comments and specs must describe why — the reasoning that code cannot encode.workflow-discipline

Don't Automate the WorkaroundIn legacy and bureaucratic domains the stated process is often itself a workaround for another broken process — automating it with an agent entrenches the dysfunction at speed; ask whether the workflow should still exist before you accelerate it.intentworkflow-disciplinemeta-principles

Earn the InterruptionHuman attention is scarce, so an agent should not interrupt merely because it hit a normal checkpoint — it should interrupt only when human judgment is now the bottleneck; insert humans at specific edges (risky deploy, unclear intent, high-cost loop, compliance exception, repeated nonconvergence), not everywhere.human-factorsorchestrationworkflow-discipline

Embed Guidance in Tool OutputInstead of front-loading every project gotcha into a giant instructions file the model forgets, put the guidance in the outputs of scripts and tools — error and success messages — so the reminder appears exactly where and when it is relevant.context-engineeringtool-design

Encode Multi-Step Work as a RecipeA conversation is a trace of what happened once — a recipe is a specification that can be replayed, inspected, versioned, and recovered from.reliabilityworkflow-discipline

Eval-Driven DevelopmentBuild evals before building features — you cannot improve what you cannot measure, and you cannot trust improvements you have not defined a measurement for.verificationworkflow-discipline

Evals Are a Standing Capability, Not a One-Time GateAn eval suite that runs once at launch and is never run again is not an eval suite — it is a historical document that measured a system that no longer exists.verification

Evidence Before AssertionsA claim about a system's state is not knowledge — it is a hypothesis. Run the verification command and read the output before making any assertion about whether something worked.verificationreliability

Fail Loud Over FallbacksA fallback that hides a failure teaches the system nothing and costs the same as the failure it replaced — prefer a loud stop over a silent substitution.reliability

Fail-Loud HarnessesBuild harnesses that fail visibly — a harness that silently substitutes its own answer hides the agent's failure from you.meta-principlesreliabilitytool-design

Find the Hard StuffDeliberately seek the genuinely difficult parts of a problem — where the real work lives is almost never where it looks like it should be.taste

Finishing Is the Bottleneck, Not StartingModels make starting easy — a plausible 80% appears fast — but plausible work is not finished work, and the final 20% consumes most of the attention and risk; the real metric is convergence — work that reliably reaches an accepted, shipped, observed state.workflow-disciplinereliabilitymeta-principles

Fix Reliability Before FeaturesFlakiness, not capability, is the enemy — a system that only sometimes works is harder to improve than one that reliably fails.meta-principlesreliability

Fix the Root Cause, Demand a DiagnosisAgents tend to patch the visible symptom without pulling the thread to the cause — for recurring, production, or architecture-level issues, require a diagnosis artifact that explains the root cause before any fix is accepted.reliabilityverificationworkflow-discipline

From Personal Workflow to Shared Operating ModelIndividual engineers develop powerful personal workflows that don't compose into a team operating model — and the best local agent answer can violate strategy or shared taste; an organization needs shared artifacts, review norms, observability, and patterns.meta-principlesworkflow-disciplinehuman-factors

Gate the Phases ExplicitlyDeclare a hard transition between design and build, and between build and ship — phases that blur produce work that cannot be reviewed or rolled back.workflow-discipline

Gene Transfer — Port the Behavior, Not the CodeThe reusable unit between systems is rarely a whole tool but a trait — a review prompt, a validation loop, a context-file structure, a CLI affordance — so separate phenotype from organism and transfer what's valuable; at system scale this becomes semantic porting of behavior across stacks.knowledgetool-designmeta-principles

Give Agents the Ability to Verify Their Own WorkAn agent that can detect its own errors is orders of magnitude more reliable than one that can only produce output — build verification into the agent before shipping it.verificationreliability

Good Pile, Bad PileIn domains where stakeholders say "it depends," stop asking for abstract rules and start collecting labeled examples in the domain's own language — good, bad, acceptable-but-awkward, compliant, risky — because examples become intent training data agents can learn the dependency structure from.intentverificationknowledge

Governance Is Infrastructure, Not a DocumentIn agentic organizations governance can't be a policy doc that sits outside the workflow — it must be embedded in tools, prompts, validators, logs, and approval edges; the common substrate is an LLM proxy/gateway that logs usage, tracks cost, enforces policy, and provides observability.human-factorsobservabilityorchestration

Hand Off With a Clear Stop ConditionAn agent without a stop condition will find one itself — and it will not be the one you wanted.workflow-disciplineorchestration

Hang Up, Call BackWhen an agent conversation goes down the wrong path, don't course-correct — rewind history and re-prompt so the model never sees the bad idea.context-engineeringworkflow-discipline

If It's Important, Emit an EventIf a decision, state change, or outcome matters to understanding what happened, it should produce a structured event — not a log line, not a comment, not a hope that someone was watching.observability

Implement to LearnThe fastest way to understand a problem is to build a small, throwaway version of the solution.workflow-discipline

Incremental Validation: The 3-File RuleBefore processing a full batch, verify the pipeline works correctly on three representative files — catching bugs at three is dramatically cheaper than catching them at three thousand.verificationworkflow-discipline

Interview to a Spec Before ImplementingPut a skeptical interviewer in front of intent — one that asks why before how, reflects back its understanding, refuses to implement until it can restate the goal cleanly, and emits a concise spec the human can reject — turning messy intent into an executable specification.intentworkflow-disciplineverification

Invest the Expensive Model in the HarnessSpend your strongest, most expensive model on improving the harness itself — then run cheaper models through the better harness — because better orchestration is second-order leverage that makes weaker models more useful.cost-routingorchestrationmeta-principles

Just-in-Time Retrieval Over Pre-LoadingDon't fill context with everything that might be relevant — retrieve only what is needed, exactly when it is needed, and discard it after use.context-engineering

Keep Specs in SyncA spec that diverges from the implementation is worse than no spec — it actively misleads every agent that reads it.workflow-discipline

Layer Guardrails and Human EscalationBuild in multiple layers of guardrails — input validation, output checking, action limits, and explicit escalation paths — so errors are caught before they compound and humans are brought in before damage is done.orchestrationreliability

Lead With Recon Before ActionBefore taking any action that touches external state, first run a read-only reconnaissance pass to understand what exists, what constraints apply, and what the action will actually affect.orchestration

Let Agents Evaluate and Optimize Your ToolsUse an agent to test your tool set — have it attempt real tasks, observe where it selects incorrectly or fails, and feed those observations back into tool descriptions and interfaces before deploying.tool-designverification

Leverage Over EfficiencyFraming AI as efficiency asks how to do the same work with fewer people; framing it as leverage asks how the same people can build, explore, and operate at a larger scale — and leverage is the frame that matches what advanced practitioners actually do.human-factorsmeta-principles

Make Every Stage Produce an Auditable ArtifactA stage with no artifact is a stage that cannot be audited — and a stage that cannot be audited is a stage you must trust blindly.reliabilityobservability

Mark Un-Run Steps Inline for ReviewA skipped step that is not explicitly marked is indistinguishable from a completed step — make the gap visible or it will be treated as a gap that was filled.reliabilityobservability

Match Model to StageDifferent stages of a workflow warrant different model capability levels — route expensive models to complex reasoning stages and cheap models to mechanical stages, not the other way around.cost-routing

Match the Agent Metaphor to the LayerTreating every agent as a person creates confusion; treating every agent as a stateless tool loses useful continuity — choose the framing per layer — disposable process, independent critic, manager-like orchestrator, long-lived assistant, or policy-bound organizational actor.human-factorsorchestrationmeta-principles

Match the Topology to the WorkChoose your agentic topology deliberately — from one-shot prompt to slash-goal loop to skill to graph to hierarchy to studio — by the cost of failure, clarity of intent, size of the work, and degree of repeatability.orchestrationworkflow-disciplinemeta-principles

Mine for ContradictionsContradictions are high-signal — they reveal where an organization has changed its mind without updating its artifacts or has never reconciled competing goals; run a detector across transcripts, specs, and decisions that surfaces conflicts for discussion rather than auto-fixing them.knowledgeintentverification

Mine Transcripts Into Structured KnowledgeMeeting transcripts, session logs, and Slack threads are raw material, not output — record, then transform them into topic clusters, entity graphs, contradiction lists, action repositories, takebooks, and persona material; recording everything without transformation produces storage, not knowledge.knowledgecontext-engineeringobservability

Namespace Tools, Write Unambiguous DescriptionsGroup tools under a clear namespace and write descriptions that tell the model exactly when to call this tool and when to call a similar one instead — ambiguous tool selection is a primary source of agent errors.tool-design

Nontechnical Owners, With GuardrailsAgentic tooling lets people with clear intent and domain knowledge ship software they could never have built before — but ownership by nontechnical builders only works when surrounded by scaffolds, review loops, acceptance evidence, rollback, and coaching.human-factorsreliabilityworkflow-discipline

Parallel Attempts at One TaskBecause a model is non-deterministic, one attempt is a single die roll — for a hard or quality-sensitive task, run several implementations of the same task in parallel from one checkpoint, then pick the best or splice the strongest parts of each.orchestrationcost-routingworkflow-discipline

People Edit Better Than They AuthorMost stakeholders cannot write a clean requirement, but shown a concrete artifact they can instantly say what's wrong — so elicit intent by giving people something to react to, because cheap software has made a throwaway artifact the fastest way to extract intent.intentworkflow-disciplinehuman-factors

Persist Environment FactsFacts about the environment — directory layout, versions, credentials, topology — belong in files the agent can re-read, not floating in context that disappears on reset.context-engineering

Prefer Search/Filter Over List-EverythingGive agents a search or filter interface rather than a tool that returns a full list — agents that list everything then scan results are burning context on data they'll mostly discard.tool-designcontext-engineering

Prove the Pipeline on 2 Before 20Run the full pipeline on two items end-to-end before scaling to the full dataset.workflow-disciplinereliability

Pull Models From Provisioned RolesReference abstract capability roles in code rather than model names — let external configuration map roles to models, so routing decisions survive model changes without code changes.cost-routing

Put Deterministic Rails Around Model JudgmentThe strongest agentic workflows interleave deterministic steps and model-mediated steps so neither layer must be perfect — the model translates intent into checkable state, and scripts, tests, and validators catch drift.orchestrationreliabilityverificationtool-design

Rebuild OftenRegular rebuilds from scratch catch accumulated drift that incremental changes normalize.workflow-discipline

Redesign the Work, Don't Fill Human-Shaped SlotsReplacing each human role with an agent-shaped equivalent preserves old inefficiencies — and measuring maturity by percent of code generated measures the wrong thing; redesign the work around what models and tools can now do.meta-principleshuman-factorsworkflow-discipline

Repository Shape Is Cognitive ArchitectureA repository is not just code storage — it is the working memory and context boundary of an agentic workflow; too broad and it poisons context, too narrow and the agent starves, so repo shape is a design decision, not an accident.context-engineeringworkflow-disciplinemeta-principles

Requirements Are Disguised SolutionsStakeholders ask for a field, a button, a report, or a workflow change — but agents will happily implement the stated solution even when it's a workaround for a broken process, so the intent layer must ask what the request is actually trying to accomplish.intentworkflow-discipline

Run Independent Agents in ParallelWhen two or more tasks have no ordering dependency and no shared mutable state, run them in parallel agents rather than sequentially — wall-clock latency is the only cost that matters.orchestration

Run Real Workloads in Their Proper EnvironmentAgents that run workloads in simulated or sandboxed environments discover only simulated problems — run the real thing in the real place if you want real answers.observability

Scope and Expire MemoryA single global memory pool causes context poisoning, obsession, and stagnation — agents cling to discarded architectures and stale assumptions; default to scoped, expiring memory (project-, role-, task-specific), because forgetting is a feature, not a bug.context-engineeringknowledgereliability

Self-Modifying Context Needs ReviewLetting an agent edit its own persona or instruction files is powerful and risky — it can correct repeated bad habits but also lock in bad lessons, please the user instead of solving the problem, or drift unaudited; require reflective changes that cite evidence and pass review before becoming canonical.knowledgereliabilityhuman-factors

Separate Merge From ExposureFeature flags are one of the most important controls for agent-generated work because they separate merging code from exposing it — default risky changes to staged exposure with explicit rollout criteria and automated rollback, so merge no longer means everyone is affected.reliabilityworkflow-disciplineorchestration

Separate the Proposer From the AuthorityThe entity that proposes an action is the worst authority to approve it — route approval through a role with no stake in the proposal's acceptance.reliabilityorchestration

Shadow the User, Then Transform the TranscriptTacit knowledge — workarounds, local norms, exception paths, institutional memory — lives in how people actually work, so sit with users and record them; agentic tooling doesn't remove the observation, it accelerates what happens after it.intentknowledgehuman-factors

Shape the Workflow as an AttractorDesign the workflow so the correct outcome is its stable equilibrium — the state it converges toward from a wide range of starting points and returns to after the model drifts — and judge success by that convergence, not by whether a step reported done.orchestrationworkflow-disciplinemeta-principles

Share Full Context; Every Action Carries DecisionsWhen delegating to a subagent or spawning a parallel worker, include everything the agent needs to make good decisions — the goal, constraints, prior findings, and any decisions already made.orchestration

Simulated Review SquadBefore work reaches real users, run it past a panel of simulated perspectives — expert and novice users, compliance, support, executive sponsor, skeptical engineer, accessibility, security — built from real transcripts and tickets, to surface objections early and cheaply.intentverificationreliability

Start With the Least Agentic ThingMany problems that look like multi-agent workflows need only one well-crafted LLM call.meta-principlesorchestration

State the Target, Not the ProhibitionTelling a model what not to do activates the very concept you're suppressing — 'list the planets but not the moon' tends to produce the moon — so phrase instructions as a positive description of the target, and enforce hard prohibitions with a deterministic check rather than louder "do NOT".context-engineeringreliability

Supervise Many Agents Through One InterfaceAs agents multiply, the limiting factor is the human's ability to see them — build the supervision surface — a dashboard of what's running, stuck, done, costly, or waiting; status exhaust; an attention firewall; a review queue; a cost monitor; and the ability to resurrect and hold old sessions accountable.observabilityhuman-factorsorchestration

Test What the Product Does, Not HowTests that describe externally observable behavior survive refactoring; tests that describe internal implementation become a maintenance tax that grows with every change.verification

Tests Are Code — Update Them First on API ChangeWhen an API changes, update its tests before updating its implementation — the tests are the specification, and updating them first forces you to define what the change means before you build it.verificationworkflow-discipline

The Agent's Struggle Is a SignalWhen an AI agent starts thrashing on a codebase — missing duplicated call sites, running out of context mid-change, making excuses about failing tests — read it as an early warning that maintainability is decaying, and refactor, instead of just fighting the agent.observabilitytaste

The New Literacy Is Operating, Not PromptingWhen the machine writes most of the implementation, the skill that matters shifts from typing code to steering, decomposing, inspecting, validating, and governing agentic workflows — train for operating, not for prompting tricks.human-factorsworkflow-disciplinemeta-principles

Two-Phase Build: Architect Then BuilderSeparate design from implementation by running an architect agent first to produce a complete specification, then a builder agent to implement it — never conflate the two roles in one pass.orchestrationworkflow-discipline

Use a Second Agent as AuditorThe agent that did the work is the worst possible choice to verify the work — put a second agent in the audit seat with no stake in the original claim.reliabilityorchestration

Use Session History as Primary EvidenceWhen something went wrong — or right — in a prior run, the session transcript is the primary evidence; reading it is the first act of diagnosis, not a fallback after other approaches fail.observability

Use Sub-Agents as Context SinksDelegate detail-heavy work to sub-agents so the orchestrator's context stays clean — the sub-agent absorbs the complexity and returns only a summary.context-engineeringorchestration

Verify Independently — Agents Fake "Done"An unobservable step is effectively unrun — treat "complete" as a claim that must be independently verified.meta-principlesreliability

Video-Driven DevelopmentRequire agents to produce a before-and-after video of the real user scenario — a video sets a higher bar than a textual claim and forces the agent to exercise the actual product surface, not just the code; the rule is show, don't assert.verificationreliabilityobservability

Work Backward From the Shipping DockStart from where work becomes real — production, approval, operator adoption, user comprehension — and ask what proof would convince you it's ready; the proof system then shapes the workflow, instead of asking "can the model build this?"workflow-disciplineverificationreliability
