---
url: https://www.infoq.com/articles/when-spec-driven-development-pays-off/
date_fetched: 2026-09-13
---

### Key Takeaways

- With AI coding assistants, the bottleneck has moved from code writing to code verification.
- A specification baseline did not make reviewers better bug-finders. It turned drift review into a contract-anchored, attributable, higher-confidence activity, but at a measurable time and cost.
- The approach of authoring the specification and then implementing from it in a fresh generation step moved the outcome, by treating the spec as a governing artifact rather than inline prompt text.
- On easier tasks, the dramatic gains people attribute to "specifying first" are largely a reasoning effect in disguise. If you are going to claim specification prompting improves quality, control for reasoning first.
- Specification governance is the best investment in one specific area: hard, multi-constraint work built by a capable-but-imperfect model.
 

## Verification Is the New Bottleneck

AI coding assistants stopped being a novelty and became infrastructure. In 2026, most engineering teams use coding assistants weekly. AI authors a large and rising share of production code. AI generates code across the whole lifecycle. The uncomfortable part is that AI raised the volume of code without raising anyone’s confidence that the code is correct, secure, or aligned with what was actually intended. Controlled studies report real productivity gains, but also slowdowns and quality problems under production conditions, as well as a steady stream of empirical work showing that AI-generated code reaching production with security weaknesses, familiar bug patterns, and quiet behavioral drift.

So the bottleneck moved. It used to be writing; now it is verification, which reframes the hard question as a governance question rather than a model-capability one: Who is accountable for AI-generated behavior, how do you detect when it diverges from intent, and how do humans and models split the labor of oversight?

This matters in 2026 Q2, not in the abstract, because the obligations are landing now. The EU AI Act’s high-risk provisions demand risk management, record-keeping, and meaningful human oversight. ISO/IEC 42001 asks for a documented AI management system with controls and audit trails. The NIST AI Risk Management Framework organizes the same expectations under Govern, Map, Measure, and Manage. If a growing share of production code is AI-generated, simply saying we review the output carefully provides little evidence of how risk management, record-keeping, and human-oversight controls are actually implemented.

The popular response is to review the spec, not just the code. This approach treats the specification as the contract the AI must satisfy, reviews up front, and holds the generated code accountable to it. I believe in that framework, but I wanted to know whether it survives measurement, so I built a study around the one act at its center, a human reviewing AI-generated code against an approved baseline.

The honest result changed how I argue for the whole idea. This is the reason for this article. A specification baseline did not help reviewers catch more bugs. Rather, the baseline made the bugs they caught accountable. Knowing that distinction is what lets you decide when the cost of using AI is worth paying.

The findings below come from a study of mine accepted at GAISS 2026. They are preliminary and directional with small samples and a few tasks. I will be specific about the limits throughout.

## Treating the Specification as a Governance Baseline

A specification is the written, agreed description of what a piece of software must do before anyone builds it, including the requirements, the interfaces, and the behavior that can be tested. In this study a specification has three layers: the specification itself (the business requirements and rules), a high-level design (HLD) that defines the components and their interfaces, and a low-level design (LLD) that pins down the concrete invariants each method must uphold. For example, a specification for a money-transfer service states a business rule such as "a transfer must never leave an account balance negative".

The specification’s HLD defines a TransferService interface with a transfer operation (i.e., from, to, and amount). The specification’s LLD makes that testable as an invariant: In other words reject the transfer and emit no ledger entry when amount exceeds available balance. Those clauses are exactly what a reviewer later checks the AI-generated code against.

To make this concrete, here is one thread of the actual baseline used in the study, traced through all three layers. At the specification layer, a business requirement states that a transfer must move funds between accounts atomically, apply at most once per idempotency key, and never create or destroy money. At the HLD layer, those requirements become a contract.

A transfer operation (including `from_id`, `to_id`, `amount`, and `idempotency_key`) on the Bank component, with transfers, validation, and an ordered audit history identified as distinct subsystems. At the LLD layer, the same requirement is pinned down as a testable method contract. Look up both accounts (raise on unknown). If the idempotency key was already applied, return the prior result without moving funds again. Next, verify sufficient funds or abort with no change to either account. Then debit the source and credit the destination as a single unit and append a history record to both. Each clause maps to a named invariant a reviewer can cite: `transfer_idempotent`, `transfer_atomic_on_fail`, `transfer_moves_funds, and assets_conserved`. The full baseline defines twenty such invariants across core money operations, transfers, interest, daily limits, statements, and account lifecycle. That numbered invariant list is the drift contract the reviewer checks against the generated code.

Before presenting the results, here is the governance model under test. The core move is to treat the specification not as prompt decoration but as the governance artifact, the reviewed, approved, versioned contract that every control point and every accountability decision refers back to. It rests on three principles: govern the input before the output (a reviewed spec is cheaper to fix than generated code), make the baseline explicit and auditable (the approved specification plus high-level and low-level design form one reference baseline), and keep a human in the loop for judgment, not volume (automation flags divergence, while a named human decides what is correct).

That approach produces five lifecycle control points, each emitting an artifact the next one consumes, as summarized in Table 1.

| # | Control point  | What happens  | Artifact produced | 
| 1 | Specification authoring | Write the spec, HLD, and LLD before any code | Draft baseline | 
| 2 | Specification review gate | Reviewers approve completeness, interfaces, constraints, testable behavior | Approved baseline (with approver + timestamp) | 
| 3 | Guided generation | The model generates from the approved documents | Generation record (baseline version + model) | 
| 4 | Drift detection | Compare generated code against the baseline; flag divergence | Drift log (each divergence + violated clause) | 
| 5 | Reconciliation | A human resolves each drift; fix the code or update the design | Reconciliation record (what changed, by whom, why | 

**Table 1. The five lifecycle control points of specification governance, each emitting an artifact the next consumes.**

The important part is not just that these five control points exist, but that each one leaves something concrete behind. The approved baseline captures what humans agreed the system should do. The generation record ties the code to that specific baseline. The drift log records where the implementation diverged, and the reconciliation record captures how a human resolved that divergence. Together, those artifacts turn the workflow from “we reviewed the AI output” into something that can actually be traced, explained, and audited.

The resulting architecture, as shown in Figure 1 below, is an iterative loop rather than a one-way prompt. A baseline is authored and approved, the model generates against a specific version, drift is detected against that same version, and a human reconciles each meaningful divergence. A change to the design returns to the review gate; a code defect returns to generation or implementation. The records produced along the way form the audit trail.

[Click on the image to enlarge it]

**Figure 1. Specification-driven governance loop for AI-generated code (Source: created by author). **

The point of this layout is that the audit trail every governance regime asks for is not extra work bolted on. Rather, it falls out of the loop as a by-product. Each AI-generated behavior traces to an approved requirement and an accountable human decision. That is also where accountability gets named explicitly: The model is Responsible (R) for generation but never Accountable (A), the two core roles in a RACI model (i.e., Responsible, Accountable, Consulted, and Informed). Accountability always rests with a human. That is the operational meaning of “meaningful oversight”. It is the part conceptual governance frameworks usually leave undefined.

## The Drift-Review Study: Recall vs. Attribution

The framework’s central human act is drift review, judging whether AI-generated code diverges from the agreed design. So I ran a controlled, within-subject study on exactly that act.

I evaluated that review on two dimensions: recall, meaning how many of the real drifts in the code a reviewer successfully identified, and attribution, meaning whether each finding could be tied back to a specific approved requirement or named invariant rather than simply reported as something that looked wrong.

The system under study is a multi-account banking service, a deliberately small but genuinely regulated financial domain, where correctness, validation, spending limits, interest accrual, account statements, and a complete audit trail all matter. The system exposes an API a real core-banking system would recognize with `open_account`, `deposit`, `withdraw`, `balance`, `transfer`, `history`, `set_daily_withdraw_limit`, `accrue_interest`, `statement`, `total_assets`, and `close_account`. Two independent implementations (services A and B) were generated so results would not hinge on a single codebase. This domain was chosen because it concentrates the properties that make AI-generated code risky. Money must be conserved, transfers must be atomic and idempotent, overdrafts and daily limits must be enforced and every balance change must be auditable. This is exactly the kind of multi-constraint, high-stakes logic where a plausible-looking but subtly wrong implementation causes real harm.

Five reviewers, each with three-to-ten years of experience, did two independent reviews of real AI-generated banking services, in a counterbalanced 2×2 design. In the Baseline condition, a reviewer received the approved specification, HLD, and LLD, and reviewed the code against it. In the Code-only condition, they received only the public API and had to judge correctness. This is precisely today’s default AI-code-review workflow, in which a human judges generated code with no approved contract. Each service appeared in both conditions across the pool, so reviewer-level and service-level confounds cancel out. Reviewers worked alone without running the code, without an AI assistant, and without web access. Ground truth was eleven adjudicated drifts in one service and ten in the other.

The findings were established by inspecting the source, because a single test suite missed real defects the reviewers correctly found. The banking services under review were implemented in Java and Python, with JUnit and Mockito for the test suites that established ground truth. Generation and the automated-reviewer replication used a mix of frontier models, including Anthropic Claude Opus 4.8, OpenAI GPT-5.2, DeepSeek V4, and Google Gemini 3.1. The specification, HLD, and LLD were authored as a single versioned Markdown baseline. The statistical tests (i.e., Wilcoxon signed-rank, Mann-Whitney, and Fleiss' kappa) were implemented in Python using only the standard library.

A few details on how the study was run. Ground truth was established by adjudicating each service against its twenty-invariant baseline by source inspection, eleven real drifts in service A and ten in service B, because a single executable test suite missed defects the reviewers correctly found. Each reviewer report was scored on two axes: recall (distinct real drifts reported divided by the adjudicated real-drift count for that service) and attribution (whether each finding was tied to a specific named invariant rather than a generic "something looks off"). Inter-rater agreement used Fleiss’ kappa on the found/not-found vectors and the paired Baseline-versus-Code-only contrast used a Wilcoxon signed-rank test, because every reviewer did one review of each kind.

The automated replication reused the identical 2×2 design and the same adjudicated ground truth, with large language models placed in the reviewer seat for ninety machine reviews, scored by the same recall and attribution definitions and compared with a Mann-Whitney U test. The generation-side experiments, which produced the delivery-mode and chain-of-thought results, were separate. They compared arms such as direct generation, chain-of-thought (reason first, no specification), a single prompt containing spec-then-code, and a staged arm that authored a SPEC.md and then implemented from it in a fresh step, across families ranging from single-function tasks to the full twenty-invariant bank service, on a mix of strong and deliberately weak models so any specification effect could be isolated from raw model capability.

Here is the headline, which is a trade-off rather than a win, as shown in Table 2.

| Measure     | Baseline     | Code-only     | Difference (Δ)    | P-value (p) | 
| Mean recall | 0.525 | 0.518 | +0.007 | 0.69 | 
| Mean attribution rate | 0.81 | 0.00 | +0.81 | 0.043  | 
| Mean drifts found | 5.6 | 5.4 | +0.2 | N/A | 
| Mean time (min) | 48.4 | 26.7 | +21.7 | 0.043 | 
| Mean confidence (1-5) | 4.2 | 3.4 | +0.8 | 0.068 | 

**Table 2. Baseline vs. Code-only drift review (Five reviewers, within-subject; paired Wilcoxon).**

### Recall Did Not Improve

Reviewers caught statistically indistinguishable numbers of drifts with and without the baseline (0.525 vs. 0.518, p=0.69). I state this plainly because the intuitive selling point of spec-driven governance is that it makes reviewers better bug-finders and on this task it simply did not.

### Attribution Is Where the Effect Lives

I define a finding’s attribution as whether the reviewer tied it to a specific violated requirement, a named invariant, rather than a generic “something looks off”. Baseline reviewers attributed eighty-one percent of findings to a named clause. Code-only reviewers, with no contract to point at, attributed zero percent (p=0.043). Reviewers found the same defects, but they become ownable (i.e., tied to an approved requirement) only when a baseline exists. Code-only reviewers kept hedging that they "could not tell whether the behavior was intended". That sentence is the whole governance gap in a single line.

The baseline also raised confidence (4.2 vs. 3.4, p=0.068) and cost significantly more time (about forty-eight vs. twenty-seven minutes), because reviewers walked the invariant list method by method. That time cost is real and I will come back to it.

For an oversight audience, the recall null is the honest finding and the attribution gain is the contribution. A specification baseline did not make reviewers better bug-finders. Instead, it turned drift review into a contract-anchored, attributable, higher-confidence activity, but at a measurable time cost. That is exactly what a governance regime wants, not more findings, but findings a named reviewer can tie to an approved requirement, defend, and hand off.

## Does it Hold When a Model Reviews? A Ninety-Review Replication

Five human reviewers is a real but small signal. To check whether the pattern is structural rather than a quirk of five people, I replicated the identical 2×2 design with large language models in the reviewer seat, three strong assistants, and ninety machine reviews, scored by the same adjudicated ground truth and the same attribution definition.

This is a convergent replication in a separate population, not extra statistical power for the human claim. It measures an automated reviewer, rather than a human. I never conflate the two. What it shows is that the same structure reappears, as reported in Table 3.

| Measure | Baseline | Code-only | Difference (Δ) | P-value (p) | 
| Recall | 0.376 | 0.377 | −0.001 | 0.95 | 
| Attribution rate | 0.67 | 0.00 | +0.67 | <10⁻⁴ | 

**Table 3. Automated-reviewer replication (Three models and ninety machine reviews; MannWhitney).**

Recall is unchanged between conditions. Attribution collapses from 0.67 with a baseline to exactly 0.00 without one, consistently across all three models. That 0.00 is structural, not a scoring artifact. With no approved contract, there is no requirement to cite, for a human or a model alike. An LLM reviewer, like a human reviewer, needs the approved specification to make findings attributable. This need also tells you where these increasingly popular LLM code-review tools fit: They are a fine pre-screen step in the loop, but without a governed baseline they can flag issues and still not tie any of them to a requirement or produce an accountability trail.

## Why Delivery Beats Presence

If the review side says a baseline buys accountability, the generation side says something equally counter-intuitive: How you deliver a specification matters more than whether one exists.

On a twenty-invariant banking service, one arm asked the model for "a specification, then the code" in a single prompt. That was indistinguishable from just asking for code directly. Both landed at 23.8 percent on a weak model. A staged arm, author the specification, then implement from it in a fresh generation step, nearly doubled the pass rate to forty-five percent and halved the builds that failed to assemble. These were the same words with two deliveries. Only the one that treated the spec as a governing artifact rather than inline prompt text moved the outcome. (This trend of p≈0.18 at n=4 carries a second-pass confound discussed below).

Now comes the part most treatments skip. When "specify first" appears to work wonders on simple tasks, you have to ask whether that is the specification helping or just the extra deliberation the specification forces. So I added a control: Ask the model to reason about edge cases first, but write no specification at all. Across ten single-function families on two strong assistants, that chain-of-thought arm captured almost all of the apparent spec-first gain. For example, fifty-nine percent (just write code) to ninety-five percent (reason first) to ninety-two percent (spec first), with spec-versus-reasoning not significant. The takeaway is deflationary and important. On easy tasks, the dramatic gains people attribute to "specifying first" are largely a reasoning effect in disguise. If you are going to claim specification prompting improves quality, control for reasoning first.

## What it Costs and Where the Evidence Is Thin

Everything above costs something. Naming the costs is the difference between governance and cargo-culting.

The most concrete cost is time. The spec-anchored review took roughly forty-eight minutes versus twenty-seven minutes for code-only, which is a recurring, per-output labor cost that scales with the number of invariants a reviewer walks against the code surface. It is predictable rather than open-ended. It is also the most automatable part of the loop (an LLM pre-screen that attributes likely drifts and routes them for regeneration before a human looks is the obvious next step), but today it is a real tax.

There is also the up-front discipline of authoring specs, HLD, and LLD, and keeping that baseline synchronized as the system evolves. That discipline is the chief deployment barrier. It eases only as drift detection is automated.

The evidence itself has honest limits. This is a deliberately small pilot with five reviewers, two services, and one domain. At n=5, the paired signed-rank test bottoms out at p=0.043, so the "significant" results certify a consistent direction across reviewers, not a calibrated effect size. Read them as a strong pilot signal, not confirmatory proof. The confidence effect is directional. The staged-generation win carries a second-pass confound (the winning arm runs the model twice, so some of the edge may come from the extra pass, rather than the spec). The automated replication is convergent support in a separate population, not a substitute for human oversight. In addition, it could inherit model biases. I report all of this information because honest evidence hands you the knife you would use against it and because governance work that hides its own uncertainty is not governance.

## A Targeting Rule for When to Spend the Effort

Put the pieces together and you do not get "always write specs". You get a targeting rule.

Specification governance pays the most in one specific corner: hard, multi-constraint work built by a capable-but-imperfect model. On the complex banking task, the weaker model, with room to improve, gained about twenty-one points from spec discipline. The strong model, which already aced the task, gained roughly two. On trivial tasks the apparent win was mostly the reasoning effect. So, reach for a targeting rule on regulated, high-stakes, long-lived systems (.e.g., finance, insurance, and healthcare) where many constraints must stay consistent, intent must survive over time, and you already owe an audit trail to a standard or regulator. Skip it for throwaway scripts and prototypes and for tasks your assistant reliably one-shots.

When you do reach for a targeting rule, the accountability structure is what makes it real. Name the roles as a RACI, an author who writes the spec, a reviewer who approves it, the model that generates, and a reconciler who resolves each drift, with a human always Accountable even though the model is Responsible for generation. Those artifacts can provide supporting evidence for the governance, traceability, record-keeping, and human-oversight controls emphasized by NIST AI RMF, ISO/IEC 42001, and the EU AI Act.

A staged adoption path, laid out in Table 4, keeps the cost manageable:

| Phase | Focus | Concrete action | Observable outcome | 
| 1 | Pick the corner | Apply governance only to your highest-stakes, multi-constraint service | Effort spent where it pays, not everywhere | 
| 2 | Stand up the gate | Add a specification review gate (completeness, interfaces, constraints, testable behavior) before generation | Ambiguities fixed on paper, before they become code defects | 
| 3 | Make the baseline explicit | Approve and version spec + HLD + LLD as one auditable baseline | Findings become attributable to named requirements | 
| 4 | Add drift detection | Compare generated code against the baseline; route each flag to a reconciler | A drift log and reconciliation record per build | 
| 5 | Automate the walk | Introduce an LLM pre-screen to triage likely drifts before human review | The 48-minute review cost starts coming down | 

** Table 4. A staged path to specification governance that spends effort where the targeting rule says it pays.**

## Conclusion

AI made code abundant and made governance, not writing the code, the new bottleneck. The instinct is to sell specification discipline as a way to catch more bugs. The measurements say otherwise: an approved baseline did not raise recall, but it took drift review from zero percent attributable to eighty-one percent attributable, at a real and now-quantified time cost. Delivery beats presence, easy-task gains are mostly reasoning. The honest limits are worth stating out loud.

So govern the input before the output, hold the code accountable to an approved baseline, and explicitly name the human accountable for deciding how each meaningful drift is resolved. Then spend that governance where the targeting rule says it pays off: hard, high-stakes, long-lived systems built by capable-but-imperfect models, rather than a keystroke of ceremony anywhere else. Make the code abundant. Just don’t make the plan optional. Don’t pretend the plan is free.
