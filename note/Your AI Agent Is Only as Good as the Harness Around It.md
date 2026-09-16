# Your AI Agent Is Only as Good as the Harness Around It

An anonymous New Stack piece (with Oracle's fingerprints visible in the conclusion) makes the now-familiar but still-underpriced argument that an agent in production is mostly *not* the model: it is the scaffolding of tool contracts, permission boundaries, context assembly, traces, and failure tests wrapped around it. What distinguishes it from the average harness sermon is that it treats each piece as a product decision with a concrete artifact — a tool contract with an error taxonomy, a trace format, a test suite grown from real incidents — rather than as a principle to aspire to.

---

## The argument in one paragraph

The claim: most agent projects fail not because the model reasons poorly but because the interactions between the model and the surrounding systems are unengineered — and the fix is to build a harness with the same seriousness developers already bring to test harnesses. Specifically, that (1) tool errors are prompts and must be written as actionable instructions, (2) permission boundaries — not model resistance — are the actual defense against prompt injection, (3) context must be assembled by a defined, recorded path, (4) traces must answer "what exactly did the agent see," and (5) test suites built from production failures are the institutional memory that survives model upgrades. This is falsifiable: if agents with naive tool descriptions, broad credentials, and prompt-only testing ship reliably, the harness thesis is overhead. The article bets they don't, and the industry's incident record so far backs the bet.

## Key quotes

> "The model is one part of the service. The agent harness is the rest—the scaffolding the application builds around the model."

The framing sentence, and a useful one: it recasts "agent" from a model-plus-prompt into a distributed system with one probabilistic component. Everything else in the piece is a consequence of taking that sentence seriously.

> "An error message is a prompt. `ERR_422` teaches the agent nothing. `APPROVAL_REQUIRED: annual plan changes need human sign-off` tells it exactly what to do next."

The sharpest observation in the article, and one most teams still get wrong. Because the model reads tool output and acts on it, your error taxonomy is part of your prompt surface — which means error design belongs to whoever understands the agent loop, not whoever owns the API by accident.

> "You can't count on the model to resist every instruction that arrives embedded in data, so the permission boundary is a critical security boundary."

A refreshingly blunt statement of the prompt-injection posture: assume the injection succeeds, then make success cheap. Scoping each tool to its own service identity and passing user identity as a verified token — never as a model-filled parameter — is the concrete mechanism, and it's the right one.

> "Can the team determine exactly what the agent saw for a specific request? If the answer is no, a later investigation will start with guesses."

The best single diagnostic question in the piece. It converts the abstract virtue of "observability" into a yes/no test you can run on day one, before anything has gone wrong.

> "None of this is glamorous. Neither is a climbing harness. Nobody notices it on the way up, and then someone slips, and it's the only thing that matters."

The closing metaphor earns its keep because it names the economic problem: harness work has no demo payoff and only shows value in the failure case, which is exactly why it gets skipped.

## Critical analysis

The non-obvious contributions are two. First, "an error message is a prompt" collapses the boundary between API design and prompt engineering into a single discipline — most teams staff these as separate concerns and the seam is where agents misbehave. Second, the insistence that permission to *answer* is different from permission to *act*, enforced as a separate confirmation-and-check gate before any write, is a cleaner formulation than most security guidance offers.

The weaknesses are structural. This is vendor content wearing an essay's clothes: the context section spends three paragraphs building a genuine problem — permission models slipping at retrieval boundaries — and then lands, with suspicious neatness, on Oracle AI Database and its `langchain-oracledb` packages as the solution. The problem is real; the implication that it requires a converged database is not, and a reader should hold that conclusion loosely. More broadly, the piece is a checklist without economics: it never says what any of this costs, how to prioritize when you can't build all five layers at once, or what a minimally viable harness looks like for a two-person team. "Start with the tool inventory and the read/write boundaries" is the only sequencing advice offered. It also stays firmly single-agent — no delegation, no multi-agent coordination, no mention of what traces mean when three agents share one task.

What it leaves out that matters: any treatment of *how* the harness evolves. The trace example is lovely, but who reads traces in practice, how often, and what organizational role notices that the agent used `policy_v41` when `policy_v42` shipped last week? The article's own test-suite-as-institutional-memory idea gestures at this and then stops.

## Related

- [[Harness Engineering]] — This source strengthens that page's core claim that the harness, not the model, is the engineering surface, and adds two concrete artifacts it can absorb: the retryable/terminal error taxonomy and the "what did the agent see" trace diagnostic.
- [[The Agent Access Model]] — The article's permissions section is a practitioner-scale echo of that paper's thesis that BeyondCorp-era controls fail for agents; it strengthens the case for per-tool service identities and verified identity tokens while staying at a much shallower depth.
- [[Idempotency Is Easy Until the Second Request Is Different]] — The billing-tool contract here is a compact worked example of that page's argument: the idempotency key exists precisely because "an agent that hits a timeout will often just try again," putting agent behavior on the list of callers that make idempotency hard.
- [[Building an Advanced Agentic Harness]] — That page derives a harness from seven composable primitives aimed at agent capability; this source nuances it by arguing the equally important half is *containment* — contracts, permissions, and stop conditions — which the primitives catalog largely treats as given.

---
*Sources: [[raw/building-ai-agent-harness]], [[summary/building-ai-agent-harness]]*
*Last updated: 2026-09-16*
