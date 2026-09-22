# Building a Production AI PR Review Agent

A video course, preserved as an auto-transcript in a gist, that spends its first half designing an AI pull-request reviewer from first principles before writing a line of code, and its second half building it inside the author's own agent harness ("Genesis Kit"). The thesis is a single reframe: the problem is not automating review, it is **selectivity** — deciding which findings are worth a scarce senior engineer's judgment and which the machine should handle or defer. Everything downstream (four parallel specialists, confidence-gated findings, a human approval queue, retrieval grounding, an append-only event spine) is derived from that one sentence rather than assembled from a stack list. It is the most explicit public worked example of *deriving* an agent architecture, and its second half is an unusually honest recording of a harness catching its own bugs.

---

## Key Quotes

> "The problem is not to have an agent that can review your code automatically … but instead the problem is the selectivity — which findings does the agent take and which ones does a human judgment actually get spent on."

This is the whole page. Almost every AI code review project starts from "can an LLM find bugs in a diff?" and ends up with a tool that comments on everything, which is the failure mode [[Agentic Code Review]] documents at scale. Starting from selectivity makes the confidence score, the escalation queue, and the human gate *load-bearing* rather than bolted on. Compare Cloudflare's independent arrival at the same place in [[Orchestrating AI Code Review at Scale]]: their sharpest finding was that telling the reviewer what *not* to flag is where the prompt engineering value lives. Same insight, reached from the other end.

> "If you remove a component … you will see a new problem coming out of it. … Necessity is the mother of invention — we'll create that necessity intentionally."

The method, not just a slogan. Delete the AI reviewer and what's left is a senior engineer with 20 PRs and diminishing attention; the pain that surfaces tells you what to build. He runs the same subtraction on observability — never naming the word until the reader has felt the absence: an agent posts "this endpoint is vulnerable to SQL injection, confidence 60%," a developer disputes it, and with no record of which context was retrieved, which prompt version ran, or what the model returned, the finding cannot be defended, debugged, or improved.

> "The 10th review of the day is not the first."

The clearest one-line statement of why human review doesn't scale — not capacity but *variance*. Quality decays across a queue, and the decay is invisible in any metric a team actually tracks. [[Reviewing Code Is a Skill]] argues review is a trainable human craft; this is the complementary observation that even a trained reviewer degrades within a single afternoon.

> "A review agent is not a replacement for human judgment. … Human judgment is very scarce. Can we put up this particular situation so that human only focuses on where it actually requires?"

Human-on-the-loop stated as a resource-allocation problem rather than a safety compromise. This is the same argument [[Optimizing for Decision Points]] makes about designing workflows to surface taste-sensitive decisions — except here it's the primary design constraint rather than a refinement.

> "Hallucination is a feature, not a bug."

Said in passing, and immediately contradicted by the design: citations required, rationale required, confidence required, retrieval grounding required, human gate below threshold. The provocation is empty; the engineering around it is not. Judge the design, not the aphorism.

> "If you have generated one token, tell me what was the output of that. If you cannot justify the output, I mean, dude, fuck you out."

Token discipline as a personal ethic, and the reason he instructs the coding agent to use Opus only as orchestrator and spawn Haiku or Sonnet for mechanical work. The same tiered-delegation pattern as [[The Advisor Strategy]] and [[Thrifty (Tiered Delegation for Claude Code)]], arrived at from a monthly-limit budget rather than an enterprise cost model.

---

## Key Themes

- #concept **Selectivity over coverage** — "The system optimizes for surfacing findings worth a senior attention and deferring the rest — not for maximal output." The architecture's root axiom, from which the confidence field, the gate, and the escalation queue all follow.

- #pattern **Map the mess** — Before choosing any technology, document what the human does today: every micro-decision, every knowledge lookup, every fatigue point, every failure. The four specialist agents are *derived* from the four things a reviewer actually does (bring codebase context, reason across separate concerns, stay skeptical, cite evidence), not chosen because four is a nice number.

- #pattern **Precise trigger, structured output** — Not "a claim arrives" but "an email to this address containing the word *claims*." The output is not prose but a list of findings, each with `agent_type`, `severity`, `file`/`line`, `confidence`, and `rationale`. The confidence drives the gate; the rationale makes the finding auditable when the confidence is itself wrong.

- #pattern **Fan-out/fan-in with a confidence gate** — Four specialists in parallel behind an aggregator that merges, deduplicates, and scores, then routes above-threshold findings to GitHub and below-threshold or critical ones to a human queue. The planner/worker/judge shape from [[Agent Orchestration]], with the judge's output wired to an escalation path.

- #pattern **Two-direction failure analysis** — For every component, ask what breaks in the *engineering* direction (GitHub retries the same delivery, a forged payload arrives, the 10-second ack budget vs. a 90-second agent) and the *LLM* direction (hallucination, wrong retrieval slices, drift, tool timeouts). Then sort them by what you know you don't know, what you'd recognise on sight, and what you've never considered.

- #concept **Three memory shapes, one store** — Semantic (code embeddings), episodic (past reviews and disputes), procedural (team conventions), plus a time-series event spine. Jones's taxonomy in [[Agent Memory]] applied concretely — and then collapsed onto one managed Postgres (pgvector, pgvectorscale/DiskANN, TimescaleDB hypertables, continuous aggregates) on the argument that three databases means three sets of backups, connection pools, and corruption modes.

- #concept **Maturity owns the autonomy** — Start with more human involvement than you think you need and withdraw it as the system earns trust, rather than starting autonomous and recovering from expensive mistakes. Backed by a real incident: a conversational agent at a previous employer autonomously scheduled a dinner with a prospect in the Netherlands within nine hours, well inside the two-to-three-day window the team had assumed they'd have to catch it. "A system failed because an assumption failed."

- #tool **Genesis Kit** — The harness: a locked `DONE.html` spec no agent may edit, a context graph of invariants, live implementation notes so a cold session knows what exists, a milestone plan where each entry carries a demo command and a token budget, five gates, and an independent verifier agent given only the goal and invariants — never the builder's trail.

- #pattern **The verifier gets fresh context** — "It's like verifying or writing the exam paper for your own and then verifying your own exam paper. You'll automatically write those exam questions which you're already aware of." A separate agent, blind to the build log, is spawned per milestone to attempt disproof.

- #pattern **Invariants as non-negotiables, enforced at the lowest layer** — Every webhook payload passes HMAC verification; every finding carries confidence and rationale; the agent events table is append-only. The last one gets enforced in the database via a trigger, "not just convention."

- #tool **Framework escape hatch** — LangGraph chosen over Temporal for the MVP (runs in-process, first-class fan-out via the Send API, checkpoints to the Redis already present) but placed behind an abstract `WorkflowEngine` interface with run/resume/get_state, so the choice can be reversed without touching the rest of the system.

---

## Critical Analysis

**The derivation is the contribution, and it's genuinely rare.** Most agent architecture writing presents a finished diagram and reverse-justifies it. This presents the reasoning trace: here is the human process, here is what breaks when you remove it, therefore retrieval, therefore multi-agent, therefore confidence and rationale on every finding. The claim that "if you answer why it exists at all, the architecture becomes almost obvious" is overstated — plenty of well-motivated systems still have bad architectures — but as a teaching device the subtraction method works. The observability section in particular, where the word is withheld until the reader has felt the dispute-you-cannot-defend, is the best-constructed argument in the piece.

**The convergence with Cloudflare is the strongest external evidence for the design.** [[Orchestrating AI Code Review at Scale]] reports seven specialist reviewers plus a coordinator judge across 131K reviews at $1.19 average; [[OpenCodeReview]] reports per-file concurrent subagents with a comment filter pass; this course derives four specialists plus an aggregator from first principles with no production traffic at all. Three independent arrivals at "specialists plus a synthesizing judge" is real signal that the shape is right. But note what the derivation could not give him: Cloudflare's risk tiering (2 agents for trivial diffs, 7 for large ones), their 85.7% cache hit rate from shared context extraction, and their 0.6% break-glass rate all came from *operating* the system. Reasoning gets you the topology; only traffic gets you the thresholds.

**The unanswered questions are where the real difficulty lives, and the gist's own notes say so.** How do conflicting findings get resolved when security and quality disagree? The aggregator "merges and deduplicates" — that is a description of a wish, not a mechanism. What happens when the human approval queue grows faster than humans clear it? Named as a problem, met with "capacity planning" and "queue prioritization" as nouns. How do you know the system is improving? A "golden dataset and regression gate" with no methodology for building or maintaining either. Every one of these is the part that takes months in production, and the design loop that generated the architecture doesn't generate answers to them — because they are empirical questions, not derivable ones. The course is honest enough to list them, which is more than most.

**The one-database argument is technically sound and commercially compromised, and both are true at once.** The reasoning is good: four data shapes, one durable store, one backup story, one place to answer "for this PR, what did we retrieve, what did we produce, what did it cost?" — instead of stitching three systems together in Python. Postgres-as-everything is a defensible 2026 default and the wider convergence noted in [[Databases and Data]] supports it. But TigerCloud is the course sponsor, complete with a credits link and an affiliate pitch, and the vendor comparison section reads accordingly. The argument survives its sponsorship; a reader should still notice that the alternative considered was Qdrant rather than, say, plain Postgres with pgvector and no managed tier.

**Prompt injection is the gap that matters most and gets one sentence.** The trust boundary is correctly identified — the PR diff is untrusted input, and the agent reads it, reasons over it, and posts to GitHub with it. An attacker who can open a PR can put instructions in a code comment aimed at the security specialist. In a system whose entire value proposition is *selectivity*, an injection that suppresses a finding is worth more to an attacker than one that fabricates a finding, because suppression is invisible and fabrication gets disputed. Nothing in the design defends against it: the confidence gate catches uncertainty, not manipulation. This is the one place where the "what can go wrong in two directions" loop clearly failed to run to completion.

**Genesis Kit is the more transferable half, and the transcript accidentally proves it.** The design method is good but hard to copy; the harness is concrete and its value shows up on camera. The verifier — spawned fresh, given only the goal and invariants, explicitly denied the builder's trail — catches two real defects a self-reviewing agent would have blessed: a malformed-but-validly-signed payload returning 500 instead of 400, and a `TRUNCATE` path that bypasses the row-level trigger enforcing the append-only invariant. Then the quiz protocol surfaces a live config bug (`vector(1536)` in the schema, embedding dimension 256 in `.env`) that would have detonated several milestones later. That is adversarial verification earning its cost in real time, and it is the same argument [[Harness Engineering is not Enough]] and [[Load-Bearing Assumptions]] make from other directions. The locked `DONE.html` — a spec the agent is forbidden to edit, precisely because agents drift goals to suit themselves — is the cleanest small idea in the whole piece and generalizes far beyond this project. It is [[Specifications as the Product]] with an enforcement mechanism.

**The "quiz me" protocol is a real answer to comprehension debt, with a scaling problem.** Forcing the human to justify design decisions before a milestone is accepted directly attacks what [[Loop Engineering]] flags as comprehension debt and [[Human-in-the-Loop is Tired]] describes as the supervision that has become the whole job. But note what it costs: it converts the human from approver back into student, which is correct for a course and questionable for a team shipping on a deadline. The speaker never addresses whether a working engineer will actually sit through a quiz on their fourth milestone of the day — which is, ironically, exactly the fatigue argument he opened with.

**Read it for the method, not the code.** By the speaker's own framing this is a baseline to extend, not a finished system: two of roughly nine milestones are built on camera, retrieval quality is deferred to a future video, and the reliability layer exists as a plan. The transcript is also raw ASR — "idempotency" is consistently "item potency" or "unaltered key," the speaker's own name is mangled in the course blurb, and there are long digressions about dictation-software pricing. The signal-to-noise ratio is poor and the signal is worth the extraction.

---

## See Also

- [[Orchestrating AI Code Review at Scale]] — Cloudflare's production instance of nearly this architecture, with the operating numbers the derivation can't produce
- [[Agentic Code Review]] — Osmani's problem statement: the bottleneck moved from writing code to trusting it
- [[The End of Code Review]] — Monperrus's argument that mandatory human review is indefensible; this is the escalation design that follows
- [[OpenCodeReview]] — Alibaba's hybrid deterministic+agent CLI: a third independent arrival at concurrent specialists plus a filter pass
- [[Metis — ARM AI Security Code Review]] — The security-only counterpart, where deterministic reachability analysis gates the model instead of a confidence score
- [[Agent Memory]] — The semantic/episodic/procedural taxonomy this project implements and then consolidates onto one store
- [[Agent Orchestration]] — The planner/worker/judge pattern the fan-out/fan-in design is an instance of
- [[Event-Driven vs Polling Architectures]] — Why the fast-ack-then-enqueue webhook pattern is mandatory, not stylistic
- [[The Valley of Webhooks]] — The dedup/ordering/idempotency stack this design rebuilds from scratch
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE: the same context-plus-harness engineering with production reflection loops
- [[Cloud Agent Lessons from Cursor]] — The other side of the LangGraph-vs-Temporal call: Cursor chose Temporal for durable execution at 50M+ actions/day
- [[The Log is the Agent]] — The append-only event spine taken to its conclusion as the primary substrate
- [[Loop Engineering]] — Osmani's name for what Genesis Kit is: building the system that prompts the agent
- [[Harness Engineering is not Enough]] — The counterweight: why upfront planning matters more than harness sophistication
- [[Load-Bearing Assumptions]] — Surfacing the unproven claims a plan rests on; the design method here has no equivalent step
- [[Specifications as the Product]] — The locked `DONE.html` as an enforcement mechanism for the spec-as-durable-artifact thesis
- [[Optimizing for Decision Points]] — Human judgment as a scarce resource to be routed, which is this architecture's root axiom
- [[Human-in-the-Loop is Tired]] — The cost side of the approval queue this design creates
- [[Guardrails and Feedback Loops]] — Database-level enforcement of the append-only invariant as deterministic enforcement over instruction
- [[The Advisor Strategy]] — Tiered model delegation: expensive model orchestrates, cheap models execute
- [[Databases and Data]] — The convergent-database argument the single-store decision rests on

---

*Sources: [[raw/building-a-production-ai-pr-review-agent]], [[summary/building-a-production-ai-pr-review-agent]]*
*Last updated: 2026-08-21*
