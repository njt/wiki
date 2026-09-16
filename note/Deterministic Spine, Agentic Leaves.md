# Deterministic Spine, Agentic Leaves

A Microsoft Foundry blog post that names the failure mode nobody demos: multi-agent systems where every agent behaves reasonably and the process still fails, because no deterministic component owns state, transitions, approvals, or irreversible actions. Its prescription — a state-machine spine with agents as bounded, schema-contracted leaves — is a design-time architectural pattern, not an observability afterthought.

---

## The argument in one paragraph

Enterprise multi-agent systems fail most often not because a model hallucinates but because the *process itself* has no deterministic owner: the next step depends on what a model decided in the moment, from prompt wording, context contents, and tool descriptions, so the same request can take two different paths through an approval workflow. The claim, stated falsifiably: if you run the same request 100 times and the sequence of business states is not identical every time — or if you cannot name the one component that decides whether a transition is allowed — your system will produce unlogged control failures in production, regardless of model quality. The fix is architectural, not prompt-engineering: make the business process a state machine owned by deterministic workflow code, and demote agents to bounded workers inside the states, allowed to recommend, draft, and prepare but never to move, approve, or execute.

## Key quotes

> Dangerous multi-agent systems rarely fail loudly. They usually look like they are working.

This is the article's best observation and the reason the pattern matters: ticket created, handoff happened, tool called — every surface signal says healthy while the control guarantees have quietly evaporated.

> None of these are hallucinations. All of them are workflow-control problems.

The taxonomy of eight failure modes (misclassification cascade, inferred approval, parameter drift, concurrent writes, retry amplification, circular delegation, silent subagent failure, bypassed parent) is deliberately operational rather than semantic — a useful reframing away from the model-quality debate that dominates the discourse.

> if the spine cannot validate the output, that output cannot influence the next transition

The single enforceable rule of the whole pattern. Everything else — schemas, guards, idempotency keys — is machinery for making this sentence true in code rather than in intention.

> Enterprise reliability is won here, in unglamorous code.

A welcome anti-heroic note: the action plane of strict, boring, rejecting tools is where the article locates the actual engineering work, not in orchestration diagrams.

> deterministic where the business requires trust, agentic where intelligence creates value

The slogan the whole post is built to earn. It is also the part most easily quoted and least often implemented, because drawing the line requires the organisation to know which parts of its process it actually trusts.

## Critical analysis

What is non-obvious here is the diagnosis. The industry's default explanation for multi-agent failure is model unreliability, and the default remedy is a better model or a supervisor agent with stricter instructions. This article argues both are category errors: the failure modes it lists — inferred approval, parameter drift, retry amplification — are *structural*, produced by letting the model's in-the-moment decisions determine business-state transitions. The opening anecdote is engineered to make this visceral: five agents, all performing adequately, producing a 30-day standing admin grant because "manager aware" got read as consent and an unspecified duration got invented. Nothing hallucinated; the architecture lacked an owner. That framing — "the model is not your risk, your control plane is" — is the sharpest thing in the piece.

The second strength is that the pattern is grounded in converging independent guidance rather than vendor assertion. Quoting Anthropic's workflows-vs-agents distinction, LangGraph's predetermined-code-paths language, and OpenAI's handoffs-vs-agents-as-tools alongside Microsoft's own framework documentation makes the point that this boundary is now industry consensus, not a Microsoft sales angle — even though the post is unapologetically a Foundry implementation guide.

The weaknesses are the weaknesses of the genre. First, the war story is hypothetical — a composite designed for the demo-to-production arc — and the post offers no incident data on how often "inferred approval" actually occurs versus, say, plain misconfiguration. Second, the pattern has a real cost the article waves at but never prices: a fully deterministic spine is only as good as the process map it encodes, and the six-step checklist's first instruction ("map the process before you name a single agent") quietly assumes the process is knowable and stable. For the ticket-approval domain of the examples, fine; for the messy, exception-riddled processes where organisations most want agent flexibility, the spine becomes a bottleneck that people will route around — the exact exception-drift problem in different clothes. Third, the boundary between "agentic leaf" and "deterministic guard" is drawn at the schema, but schema validation of a *classification* still trusts the classifier; a wrong label that passes schema validation still cascades. The post's own misclassification-cascade row survives its pattern intact.

What is left out: any discussion of what happens when the deterministic spine itself is wrong or stale (who reviews the transition guards with the same rigour as the agent prompts?), any treatment of migration paths for systems already shipped with agents in the ownership position, and any acknowledgement that "controlled autonomy" is a marketing-friendly name for what used to be called workflow software with some ML inside. That last point is not entirely a criticism — but the post would be stronger if it admitted that the pattern's most radical demand is organisational: someone must own the process definition, in source control, and treat it as a production artifact.

## Related

- [[Handing the Agent the Whole Job]] — Strengthens this source's core distinction from the opposite direction: Bertram argues that automating steps is not automating the workflow and insists the finish line and limits be written before the first run, which is the same "deterministic owner of the process" demand stated as adoption advice rather than architecture.
- [[Your AI Agent Is Only as Good as the Harness Around It]] — Strengthens it by treating each harness piece as a product decision with a concrete artifact; this post supplies the specific artifact for the orchestration layer — a versioned, diffable workflow definition with transition guards — that the harness argument gestures at.
- [[Structural Backpressure Beats Smarter Agents]] — Shares the thesis that system structure, not model intelligence, determines reliability; this source nuances it by locating the structural fix at design time (state machines and guards) rather than at runtime backpressure.
- [[Operating Mode as Runtime State]] — Complicates it productively: both insist governance live in the runtime rather than in prompts or memory, but the O'Reilly contract for exception modes (scope, authority, expiry, status as authoritative input) addresses the temporary-elevation cases this post's rigid spine handles least gracefully.

---
*Sources: [[raw/4550068]], [[summary/4550068]]*
