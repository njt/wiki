# Smart Models, Dumb Pipes

Braydon McCormick argues that LLMs are not question-answering machines but judgment machines, and that system design should follow the same principle that made the internet beat the telephone network: let smart things handle judgment, let dumb things handle execution. Models own the "why"; deterministic infrastructure owns the "how" and the audit trail. The essay reframes the integration question from "how do we add AI?" to "where does judgment create friction in this workflow?"

---

## Key Quotes

> Models should "own the judgment work" while infrastructure "owns the mechanical work."

This is the end-to-end principle applied to AI systems. Intelligence belongs at the edges (the model making decisions), not in the network (the infrastructure). It is a clean separation of concerns that has the same theoretical elegance as the original internet design argument — and probably the same practical messiness when you try to implement it.

> Rather than "how do we add AI?", ask: where does judgment belong in this workflow, and what should deterministic systems simply execute and record?

The strongest reframing in the piece. Most AI adoption starts from the technology ("we have a model, where do we put it?") instead of from the workflow ("we have a judgment bottleneck, what resolves it?"). Starting from friction points rather than capabilities is the difference between useful integration and demo-ware.

> Human friction points — slow decisions, inconsistent judgment, high-cost errors — reveal the actually valuable intervention sites.

This is the practical test. If you cannot identify where judgment is slow, inconsistent, or expensive in a workflow, you do not have an AI problem — you have a process-understanding problem.

## Key Themes

#concept #architecture #orchestration #separation-of-concerns

**The Telecom Analogy.** The internet's "dumb pipes, smart endpoints" architecture won over the telephone network's "intelligent network" approach because it pushed complexity to the edges where innovation could happen without coordinating with the pipe operator. McCormick maps this onto AI system design: the model is the smart endpoint, the infrastructure is the dumb pipe. It is a compelling frame, though it glosses over the fact that the internet eventually re-centralized through CDNs, cloud providers, and platform monopolies. History suggests "dumb pipes" stay dumb only until someone figures out how to monetize the pipe.

**Model-Mediated Architecture.** Three concrete implementations demonstrate the pattern at different scales: an Expert Panel Simulator (public, sequential expert deliberation), DraftForge (private, production content systems), and Lodestar/Meridian (private, tiered model assignments matched to judgment complexity). The tiered model assignment idea is the most interesting — not every judgment call deserves the same model weight. This aligns with [[Scaling Long-Running Agents]]'s finding that different models suit different roles (planning vs. execution vs. judgment).

**Judgment as the Scarce Resource.** The piece shares DNA with [[Specifications as the Product]]'s thesis that the valuable artifact is the decision (spec), not the execution (code). McCormick applies this more broadly: in any workflow, the judgment layer is the bottleneck worth investing in. The execution layer is commodity infrastructure.

## Critical Analysis

The "smart models, dumb pipes" framing is elegant and directionally correct, but it under-specifies the hard problem: how do you validate model judgment? In network architecture, a dumb pipe either delivers the packet or it does not. In model-mediated systems, the model can deliver confident, well-formatted, completely wrong judgment — and the dumb pipe will faithfully execute it. [[Harness Engineering]]'s feedforward/feedback taxonomy exists precisely because judgment validation is where these systems actually break.

The telecom analogy also has a hidden failure mode. The internet's dumb-pipe architecture worked because the endpoints (computers) were fully programmable and could be updated independently. In McCormick's architecture, the "smart endpoint" is a model you do not control — it changes with every provider update, its failure modes are opaque, and its judgment is non-deterministic. The original end-to-end argument assumed endpoints you could reason about. LLMs are endpoints you can only statistically characterize. That is a material difference the analogy papers over.

The three implementation examples are tantalizing but two are private, which makes the pattern hard to evaluate. The Expert Panel Simulator is public and interesting — sequential expert assembly is a specific instantiation of the planner/worker/judge pattern that [[Agent Orchestration]] tracks across multiple independent implementations.

Where the essay is strongest is in the reframing: "where does judgment belong?" is a better question than "where should we put AI?" Where it is weakest is in assuming the judgment/execution boundary is clean. In practice, execution surfaces information that changes judgment (the [[Guardrails and Feedback Loops]] self-tightening loop), and the boundary between "deciding what" and "doing how" is where most real system complexity lives. [[Compound Engineering]]'s insight — that the system that produces code matters more than any individual piece of code — applies here too: the system that routes between judgment and execution is the actual product, not either layer alone.

## Cross-Links

- [[Harness Engineering]] — The feedforward/feedback taxonomy is what validates the judgment McCormick wants models to own
- [[Guardrails and Feedback Loops]] — "Linters beat prompts" is the deterministic enforcement layer McCormick's dumb pipes need
- [[Specifications as the Product]] — Same thesis at the code level: judgment (specs) is durable, execution (code) is disposable
- [[Scaling Long-Running Agents]] — Independent discovery that different models suit different judgment roles (planning vs. execution vs. judgment)
- [[Agent Orchestration]] — Planner/worker/judge is the concrete implementation of McCormick's smart/dumb separation
- [[Compound Engineering]] — The system that routes between judgment and execution is the actual product
- [[The Dark Factory is a DOT File]] — The pipeline spec is the judgment layer; the factory code is the dumb pipe
- [[Feedback Loop is All You Need]] — The self-tightening loop that McCormick's clean separation does not account for

---

*Sources: [[raw/smart-models-dumb-pipes]]*
*Last updated: 2026-05-14*
