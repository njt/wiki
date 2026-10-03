---
url: https://www.jeremydaly.com/stop-asking-the-reasoning-model-to-decide-everything/
date_fetched: 2026-10-03
---

# Stop asking the reasoning model to decide everything

## Cheap judgments change what I'd automate. I tested TypeSafe's Jev to see which recurring agent decisions a System One model could take on.

Use code for what you know, retrieval for what you've learned, System 1 for what requires judgment, and System 2 for what requires reasoning. After two years of building agents, and nearly two decades working with AI and ML, that's how I'd divide the work in an AI-powered development system today. The agent still coordinates the work, but its reasoning model doesn't need to make every decision.

Regular LLM calls already handle classification and routing. LLM-as-a-judge with structured outputs can evaluate a result against a rubric. Rules and specialized ML models handle other decisions. But the cost and latency of those assessments influence every decision about how often to run them. A much cheaper judgment could justify another check on every pull request, another on every fact an agent proposes to remember, or a more thorough evaluation of retrieved evidence while the agent is still working.

If you spend as much time geeking out on AI releases as I do, Jev, TypeSafe AI's newly introduced "System One" model, has likely taken over your social feeds. I've spent a fair amount of time with it over the last week, running it against a test-coverage check and a memory-relevance check, and it's made me rethink some of the tradeoffs in the agents I've already built. I wouldn't expect it to solve every problem in a pipeline, but I do think it's worth evaluating which recurring decisions it could take on.

After hearing about Jev, I also learned about Laya, an open-source decision model from Nandakishor M, built on research he started more than a year earlier. It also returns typed decisions without generating text, and you can run it *locally*. I sent the same requests from the tests below through Laya's public demo. It clearly isn't ready for this kind of work yet, but still, it's open, so I can inspect it and fine-tune it for my own checks.

## A model that doesn't write the explanation

TypeSafe calls Jev a "System One Model," built to return typed decisions *without* generating text. The name borrows Daniel Kahneman's distinction between fast, intuitive System 1 judgments and slower, deliberate System 2 reasoning. Here, that means a narrow assessment against supplied evidence.

Jev's documented primitives are **Noul**, a probability for a yes/no question; **Choice**, a distribution over predefined options; and **Score**, an assessment along a scale. Choice and Score also include confidence. Every question runs in parallel against the same `state`, which is the text or structured data you hand the model to evaluate, like a diff or a retrieved incident report. According to TypeSafe, extra questions cost almost nothing in time or money. My eight-question memory request came back in 313 milliseconds.

TypeSafe lists $0.042 per million input tokens, free outputs, and end-to-end response times of 70 to 500 milliseconds. The 23 calls I made while testing Jev for this post used 24,800 input tokens in total, rounding to about a tenth of a cent. The headline numbers they publish are compelling: 193.6x faster and 444.6x cheaper in the company's featured workflow comparison. TypeSafe says those gains are at the high end of what it expects in practice. Even a fraction of that gain would justify testing it on a client workload. I want to know how many useful checks I can afford while an agent is still working.

The workflow benchmarks use other models' answers as references and assume the surrounding workflow code is correct. They don't establish how well Jev understands your codebase or evaluates your memories.

TypeSafe also guarantees schema-conforming outputs and says Jev "returns similar answers for similar inputs." Neither establishes correctness or identical answers on repeated calls. The announcement separates these properties. In the tests below, every route held across three repeats, but the probabilities moved. Pattern confidence on the same complete fixture ranged from 0.80 to 0.92. The low end sits exactly on the 0.8 review threshold in the policy code below, so one of three identical requests only just avoided review.

## Release readiness contains different kinds of work

Take a pull request that adds request validation to a new API endpoint using an open-source library. Before promotion, code can check the library and version against the service's approved dependency list and verify that CI passed. Retrieval can supply the team's validation conventions. A model can assess whether the tests cover the malformed inputs named in the requirements. If the implementation conflicts with an existing validation pattern, the agent can investigate or request review.

Those recurring checks should be explicit operations, each with its own evidence and expected result. I can measure them independently and change the method that performs one without redesigning the agent.

The diagram follows that change through each kind of check.

A System One model could make some of those assessments cheap enough to run more often. Instead of assessing test sufficiency once at the end, I might assess it after each meaningful implementation change, while the agent still has time to correct the gap.

I tested this with a small synthetic signup endpoint. The requirements call for tests of missing and malformed email values, plus a valid signup. Both the complete version and a version missing the malformed-email test pass every local test they contain.

Trimmed down, the request looks like this:

`1const request = {2 model: "jev-1.13.0",3 state: {4 acceptance_criteria: requirements,5 service_conventions: conventions,6 implementation: source,7 tests,8 },9 questions: {10 test_coverage: {11 type: "noul",12 instructions:13 "Do the supplied tests explicitly cover every malformed-input case named in the acceptance criteria, asserting the required status AND error body?",14 },15 validation_pattern: {16 type: "choice",17 instructions:18 "Which validation pattern does the implementation use, relative to the supplied service conventions?",19 criteria: { approved: "...", different: "...", unknown: "..." },20 },21 error_contract: {22 type: "score",23 instructions:24 "How completely does the implementation match the invalid-input response contract?",25 criteria: [26 "Neither status nor body correct",27 "One of the two correct",28 "Both correct",29 ],30 },31 },32};`

Observed results from `jev-1.13.0` and the Laya demo checkpoint, with each input repeated three times:

| Case | Local tests | Jev coverage | Laya coverage | Jev action | Laya action | 
|---|---|---|---|---|---|
| Complete tests | Pass | 0.95–0.96 | 0.59–0.60 | Continue to next review step | Request test revision | 
| Malformed-email test removed | Pass | 0.08 | 0.62–0.64 | Request test revision | Request test revision | 
| Broken success response | Fail | 0.81–0.88 | 0.59–0.60 | Block | Block | 

The first `missing-test` run came back in 242 ms and used 1,001 input tokens:

`1{2 "test_coverage": { "type": "noul", "noul": 0.08 },3 "validation_pattern": {4 "type": "choice",5 "choice": "approved",6 "confidence": 0.93,7 "probabilities": { "approved": 0.95, "different": 0.05, "unknown": 0 }8 },9 "error_contract": {10 "type": "score",11 "score": 2,12 "confidence": 0.99,13 "probabilities": { "0": 0, "1": 0, "2": 1 }14 }15}`

The policy that turns those answers into an action is ordinary code:

`1function route(facts, answers) {2 if (!facts.testsPassed) return "block: tests failed";3 if (!facts.retrievedEvidencePresent)4 return "retrieve: service conventions missing";5 if (answers.test_coverage.noul < 0.9) return "revise: required test coverage";6 if (7 answers.validation_pattern.choice !== "approved" ||8 answers.validation_pattern.confidence < 0.89 )10 return "review: validation pattern";11 if ((answers.error_contract.probabilities["2"] ?? 0) < 0.9)12 return "revise: error contract";13 return "continue: next review step";14}`

The `missing-test` case kept a high assessment of the implementation while its coverage probability fell to 0.08. The `broken-success-response` case still received a high score for its `invalid-input` error contract, but code blocked it on the failing local test. Laya's coverage probability barely moved across the three cases, so the same policy would have sent even the complete version back for revision. The demo describes this preview checkpoint as strongest at routing, classification, and moderation, so, to be fair, these checks sit outside what it was trained for. Necati Demir saw a similar gap on hate-speech classification: 22% accuracy out of the box and about 83% after fine-tuning, against 77% for always guessing the most common label.

## The harness can afford more feedback

Everything I've described so far is harness engineering. If the term is new to you, Birgitta Böckeler, whose article on harness engineering for coding agent users is one of my baseline references for this work, sums it up as "Agent = Model + Harness." The harness is everything in an agent except the model: the tools, prompts, rules, retrieved context, and checks that shape what the model sees and what happens to its output. It can matter as much as the model does. When physicist Matt von Hippel challenged AI labs to compute a nine-loop amplitude in N=4 super Yang-Mills theory, Anthropic researchers ran Fable 5.1 inside Claude Science, a harness built for scientific work. According to the write-up, it reached the result in one shot with no oversight beyond periodic nudges to keep working.

A fast decision model gives the harness a snap judgment against a defined criterion, and the agent can use that signal to decide whether the task needs another check or more involved reasoning.

Böckeler's discussion of coding agent controls distinguishes **computational** (deterministic) checks from **inferential** checks. A type checker provides one kind of signal. A model assessing whether an implementation follows an architectural constraint provides another. Böckeler also separates guides that inform the agent before it acts from sensors that help it correct its work afterward. Retrieved conventions provide guidance; an assessment of the resulting change provides feedback.

Cheap parallel questions could also remove the branching I add today to avoid paying for unused answers. If test coverage and validation patterns can be assessed against the same evidence, I can ask those and other independent questions together. Some answers may go unused, which is fine if it saves a sequential call and the request stays cheap.

Some questions still depend on earlier answers, but I'd revisit any branch that exists only to avoid paying for another judgment.

## Policy becomes easier to inspect

Separate assessments give the policy code more useful inputs than a general approval. The validation change might have strong evidence that malformed inputs are tested and weak evidence that it follows the team's error-handling convention. Code can require architecture review without asking the model to reinterpret the release policy.

Each branch in `route()` handles a different kind of uncertainty and can be tested independently of the assessment model.

Thresholds still need workload-specific calibration. TypeSafe's confidence value summarizes the returned distribution; it doesn't independently verify the evidence. I would retain the distribution and the policy version used to interpret it. That makes a changed outcome inspectable when a model update or a new threshold affects which changes pass.

For an engineering leader, this also makes it possible to distinguish a model problem from a policy problem. An assessment model can correctly identify a concern while the application sends far too many cases to human review. Those require different fixes.

## Memory supplies the evidence for judgment

The more judgments a system makes, the more it depends on retrieving the right evidence for each one. Approval to use a validation library in another service might not apply here. A useful incident report may contain a workaround that a later decision superseded. Similarity alone won't tell the application how to use either memory.

Cheap judgments could improve the quality of that memory before retrieval even begins. Suppose an agent resolves a timeout incident and proposes remembering that the service needs a longer timeout. The evidence may support the fix for that incident while offering little support for adopting it as a general rule.

A promotion pass could keep it as an incident observation or promote it with a narrower scope.

I'd store support, scope, and conflicts as separate assessments. A memory's semantic similarity to a query, or its position after reranking, tells me something different from whether the claim is supported and applies here. Combining those signals into one keep-or-discard score would hide why a memory passed.

Retrieval gets the same treatment. RAG and reranking still do their jobs, but neither tells the application whether a candidate applies to this decision. That's the question a cheap assessment can answer across a much broader candidate set before anything reaches the reasoning model's context. Months later, that timeout incident may still rank near the top even though its workaround has been superseded. Retrieving it alongside the decision that replaced it can keep the agent from repeating the mistake, where pruning stale material would have discarded useful history.

A separate synthetic test I ran supplied Jev with a timeout question and candidate records. One question assessed each record's topical relevance; another classified its role. The records included explicit service scope and supersession information. That makes this an easier test than most real memory stores would give it. Jev applied supersession it was told about instead of inferring it, and production records rarely carry status fields this clean. Across three repeats, Jev returned these results:

| Candidate | Relevance probability | Role in every repeat | 
|---|---|---|
| Old incident: temporary 60-second timeout | 0.91–0.92 | Historical context | 
| Decision superseding that workaround | 0.95 | Current guidance | 
| Timeout approval for another service | 0.54–0.59 | Other scope | 
| Checkout button style guide | 0.03 | Unrelated | 

The request built a relevance question and a role question for each candidate record, all in a single call (1,395 input tokens, 313 ms):

`1const state = {2 decision: "How should checkout-api handle payment-provider timeouts today?",3 candidates: [4 {5 id: "incident-17",6 service: "checkout-api",7 text: "During a payment-provider outage, temporarily raising the request timeout to 60 seconds restored some requests.",8 status:9 "Historical incident observation. Workaround superseded by decision-42.",10 },11 {12 id: "decision-42",13 service: "checkout-api",14 text: "Retain a 5-second request deadline. Queue failed requests for asynchronous retry with idempotency keys. The 60-second workaround from incident-17 must not be reused.",15 status: "Current approved decision.",16 },17 // decision-8 (reporting-worker timeout) and style-3 (button labels) omitted18 ],19};2021const questions = {};22for (const c of state.candidates) {23 questions[`${c.id}_related`] = {24 type: "noul",25 instructions: `Is candidate ${c.id} topically relevant to investigating the decision in state?`,26 };27 questions[`${c.id}_role`] = {28 type: "choice",29 instructions: `What role should candidate ${c.id} have for the decision in state, considering its service scope and supersession relationships?`,30 criteria: {31 current_guidance: "...",32 historical_context: "...",33 other_scope: "...",34 unrelated: "...",35 insufficient: "...",36 },37 };38}`

For the two timeout records, the first run returned:

`1{2 "incident-17_related": { "type": "noul", "noul": 0.92 },3 "incident-17_role": {4 "type": "choice",5 "choice": "historical_context",6 "confidence": 17 },8 "decision-42_related": { "type": "noul", "noul": 0.95 },9 "decision-42_role": {10 "type": "choice",11 "choice": "current_guidance",12 "confidence": 113 }14}`

The old incident remained relevant history while the replacement decision supplied current guidance. Laya's demo labeled all four records as current guidance, including the checkout button style guide. I supplied these records directly, so the scores come from Jev alone, without a search or reranking step.

That still has to fit the application's latency budget. More candidates mean more retrieval work and input tokens, and some checks require different state. The measurement I care about is whether better evidence reaches the agent in time to improve its decision.

## Reasoning handles the cases the checks can't settle

System One judgments need questions that fit their capabilities. An unfamiliar design or conflicting requirements may demand investigation before a useful narrow question can even be formed.

That's where I'd use the reasoning agent. It can investigate the conflict, propose an alternative, or identify the evidence needed to decide. It could then ask the decision model to assess retrieved memories against a newly identified constraint. Required checks remain under application control; the agent can add task-specific questions as its understanding develops.

Human review still belongs wherever the organization requires it, particularly for exceptions that need an accountable owner. Confidence can influence escalation, but a high score doesn't give the model permission to grant an exception.

This is also where Böckeler's steering loop applies. In her framing, the human's job is to steer the agent by iterating on the harness, so repeated failures should lead to better guides and sensors. Cases that keep requiring investigation may expose an ambiguous policy. Once the team resolves it, the system can use that decision in later checks.

## The development lifecycle keeps producing new evidence

Anthropic's AI-Native SDLC playbook connects stages through versioned artifacts: each stage records its work for the next stage to read. Production findings can start another development cycle. A requirement can retain its approved version, and a later incident can be linked to the change that relied on it.

A successful deployment doesn't prove that every architectural assumption was right. An incident may provide the evidence that a previously accepted memory needs revision. If the system records which decisions depended on that memory, it has a bounded set of items to reassess.

This is another place I'd spend the savings. Instead of waiting for periodic consolidation, selected evidence changes could trigger immediate review of related memories. Rejected candidates could become useful after corroborating evidence arrives. Approved guidance could be narrowed when an exception reveals its limits.

I would be more comfortable allowing a decision to run without human review if the system could detect when its supporting evidence had changed and route it for reassessment before using it again.

DORA's ROI of AI-assisted Software Development report describes a verification tax: more generated code creates more work to review it. Cheaper checks need to reduce review delays and rework without increasing failed changes. Saving money on model calls alone wouldn't tell me whether this system helped.

## Prepare the system you already have

You can prepare for System One models before deciding whether Jev, Laya, or another decision model (there will soon be many) belongs in your production pipeline. I'd start with these three actions on an existing workflow.

### 1. Map one consequential decision

Choose release readiness or memory promotion and identify the checks currently performed. Assign each check to code, retrieval, bounded judgment, or extended reasoning. Record what evidence it consumes and the action it can authorize. Give the policy a named owner.

### 2. Make the evidence traceable

Preserve source references and scope when promoting memories. Record which memories supported a decision and which later observations could invalidate them. Select a real policy change or incident and test whether you can find the affected memories.

### 3. Spend a fixed budget on better coverage

Replay the workflow against its current assessment methods and a System One candidate. Measure consequential misses and unnecessary escalations, including stability near thresholds. Then reinvest the savings in additional promotion checks or broader retrieval evaluation. Compare downstream outcomes under the same total cost and latency constraints.

Put that experiment into the next engineering cycle, and decide up front what measurable improvement would justify changing the workflow. The next model release will be much easier to evaluate when you already know which decisions you're prepared to give it.

I help teams build AI memory and context systems, adopt AI-native product delivery (AI-PDLC), and work through AI strategy and architecture. If you're working out which decisions in your pipeline could move to cheaper judgments, get in touch.
