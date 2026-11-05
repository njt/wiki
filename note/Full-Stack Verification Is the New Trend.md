# Full-Stack Verification Is the New Trend

Alcides Fonseca surveys a cluster of 2026 announcements — Harmonic's Aristotle, SF-Tensor's Lean formalization of NVIDIA PTX, &Truth, Theorem./Anthropic's RISC-V work, AutoResearch's WASM-to-syscall proofs — and argues they share one goal: *full-stack* formal verification across software and hardware, not just a verified kernel or a toy model. His deeper claim is economic: frontier models' proof-generating ability has erased the expertise moat that kept verification rare, and subsidized tokens make it cheap — for now.

---

## Key Quotes

> "The common factor in these announcements is the goal of achieving full-stack formalization, either across the software stack or across a combination of software and hardware. In the past, verification was performed only on a small core of the software or on a simplified model of the actual software."

The survey frame. Verification used to mean seL4: one small kernel, decades of effort. Now five separate teams claim stacks. Fonseca is cataloguing a phase change, not five products.

> "Frontier models' superhuman ability to generate proofs has since eroded this moat, and subsidized token prices make this the perfect time to conduct these experiments."

The pivot of the essay. For years Fonseca advocated liquid types precisely because you don't have to write proofs — the SMT solver does. That trade-off was about *human* scarcity of proof-writers. LLMs made proof generation cheap, so the cheap-but-weaker option loses its rationale. Note the honest hedge: this window exists partly because tokens are *underpriced*, and the experiments may not survive real pricing.

> "I am convinced that people will not read the source code and will instead rely on new techniques to establish confidence in the generated code."

The closing prediction, and the most consequential one. It converts verification from a specialist practice into the *default trust mechanism* for a world where nobody reads the code — the endpoint of the trajectory [[The End of Code Review]] and [[Agentic Code Review]] describe for review.

---

## Key Themes

#formal-verification #lean #liquid-types #proof-generation #concept

## Analysis

This is the supply-side complement to the demand-side argument Ben Congdon makes in [[The Coming Need for Formal Specification]]. Congdon says specification is where the bottleneck lands when code is free; Fonseca says the *proofs* are now free too, so the last excuse for not verifying is gone. Both are claims about the same economics: LLMs collapse the expensive side of formal methods.

The most interesting tension is Fonseca's own position shift. He spent years evangelizing liquid types as the pragmatic middle — automatic, no proof burden. He is candid that this preference was an artifact of human scarcity, and that the argument may revert when token subsidies end. That is a rare admission in this literature: the "right" verification technique is a function of inference pricing, not of the mathematics. It also quietly undermines the permanence of the whole wave — are these startups businesses, or subsidies-shaped experiments?

His language requirements are the practical bottleneck he names but doesn't solve: full-stack verification means writing production code in Lean, compiling to memory-unsafe C, debugging across the generated-code boundary. The Cedar case study (AWS explicitly rejected this path in 2024) shows how steep it is. "The next mainstream language will require both verification and speed" is the design challenge stated in one line — and nobody has shipped that language yet.

There's also a quiet implication for the dark-factory debate: full-stack verification is the strongest possible answer to the comprehension-debt problem. If a machine proves the binary refines the spec, no human needs to have read the code. That is exactly the move [[Ship Your Interpreter (Basis)]] makes — this essay supplies the ecosystem context (five teams, funding, tooling maturing) that makes Basis's approach look like a wave rather than an outlier.

**Take:** the survey is solid and the economics argument is right, but the essay underweights the specification problem. Proofs being cheap doesn't help if what you're proving is the wrong property — proving equivalence of the *deployed* code to the *model* still leaves "is the model what we wanted?" as an unverified human question. Full-stack verification closes the implementation gap, not the intent gap.

---

*Sources: [[raw/full-stack-verification-is-the-new-trend]], [[summary/full-stack-verification-is-the-new-trend]]*
*Last updated: 2026-11-06*
