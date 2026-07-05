# Causal Inference Agentic Workflow

Netflix's field-tested architecture for making LLMs reliable at Observational Causal Inference: a three-persona actor-critic loop (Principal, Actor, Critic) that forces the model through rigorous, templated causal inference workflows while keeping a human in the evaluation loop. The core insight is that scaffolding—not model capability—determines whether an LLM produces credible causal estimates. One-shot prompting with Sonnet 4.6 gave "consistently wrong answers not at all correlated with ground truth"; the same model with their scaffolded workflow recovered ground truth in 9/10 datasets. The open-source oci-agent implements this pattern.

---

## Key Quotes

> "Oversight is still needed to ensure the validity of results."

The article's thesis in five words. They're not trying to automate away judgment—they're trying to automate the boilerplate so humans can spend their attention on the parts that matter: framing questions and scrutinizing assumptions. This is the same philosophy as [[Smart Models Dumb Pipes]]: the LLM is a judgment machine operating inside a deterministic harness, not a free-range reasoner.

> "Without scaffolding, Sonnet 4.6 produced consistently wrong answers that are not at all correlated with ground truth. With scaffolding, the LLM recovers the ground truth in nine out of ten datasets."

This gap is the article's most important finding and it's buried in the evaluations section. Same model, same datasets, zero-to-hero difference driven entirely by workflow design. This validates the [[Components of a Coding Agent]] thesis that the harness matters more than the model—and extends it from coding to statistical inference. If you're building an AI system for any domain requiring rigor, the lesson is: invest in the scaffold, not a better model.

> "The baseline estimate relied on extrapolation for all members, including those with very low treatment probability — a risk of confounding by early adopter bias."

The case study section is a masterclass in why process matters more than output. The baseline model produced a "defensible regression" that was completely wrong because it didn't check for overlap. The scaffolded workflow caught the problem through a diagnostic gate (failed placebo test) and applied a known remediation (Crump-style trimming). The estimate dropped to 25% of baseline—the right answer, not the confident-sounding one. This is the [[Guardrails and Feedback Loops]] pattern in statistical clothing: deterministic enforcement beats instruction.

> "The challenge of evaluating agents without ground truth is met through combining process audits with human oversight."

OCI has no ground truth outside simulated data. You can't A/B test the counterfactual. So how do you evaluate an agent doing OCI? Netflix's answer: audit the process, not the outcome. Version-control reports, upload executable notebooks, let humans inspect every step. The same instinct as [[If AI Is Doing the Investigation, Version the Investigation]]—the investigation record IS the trust artifact. But Netflix extends it from coding to statistics, where the stakes are often higher (business strategy decisions vs. code bugs).

---

## The Architecture: Principal-Actor-Critic

The three-persona design is the workflow's structural innovation:

| Persona | Who | What |
|----------|-----|------|
| **Principal** | Human data scientist | Provides plan, identifies confounders, specifies tools |
| **Actor** | Software/LLM | Executes analysis, runs diagnostics, creates artifacts |
| **Critic** | Software/LLM | Checks blind spots, assigns credibility, suggests alternatives |

The Actor and Critic loop until the artifacts pass muster. The Principal evaluates the output—not by checking every calculation, but by inspecting the diagnostic reports and the Critic's credibility assessment.

This is structurally similar to [[The Advisor Strategy]]'s advisor-executor pattern, but inverted: in Advisors, the cheap model drives and the expensive model advises; here, the LLM is both driver (Actor) and advisor (Critic), and the human is the ultimate authority. It's also close to [[Apache Burr]]'s state-machine approach—the workflow is a predefined path through causal inference best practices, with the LLM filling in the content at each step rather than choosing the steps.

The key difference from most agent architectures: the Critic is not optional. In coding agents, review/verification is usually a separate phase or tool. Here, criticism is built into the loop as a first-class persona. The Actor produces; the Critic immediately evaluates. This tight feedback loop is what catches the early adopter bias problem before the Principal ever sees the output.

---

## Key Themes

- **#concept Scaffolding Over Prompting**: The article's central technical contribution. A one-shot prompt to a frontier model produces wrong answers; the same model inside a structured workflow produces right ones. The scaffold forces the model through diagnostic gates (covariate balance, overlap, placebo outcome, sensitivity analysis) that catch errors before they reach the output. This is the same principle as [[Tone LLM]]'s contract/adapter pattern: the LLM fills slots in a predefined schema; deterministic code handles the rest.
- **#pattern Actor-Critic Loop for Quality Assurance**: Two LLM personas—one that does, one that checks—operating in a tight loop. The Critic checks for blind spots, verifies alignment between plan and execution, and assigns a credibility level. This is more structured than generic "have the model review its own output" and more domain-specific than [[OpenCodeReview]]'s per-file concurrent subagents. The Critic has a specific checklist derived from causal inference methodology.
- **#pattern Process Audit Over Outcome Audit**: When ground truth is unavailable, audit the steps, not the answer. The workflow produces version-controlled reports and executable notebooks that the Principal can inspect and re-run. Each diagnostic is a gate: if overlap is bad, the estimate is flagged regardless of whether the number "looks reasonable." This is the opposite of vibe-based evaluation.
- **#tool oci-agent**: Netflix's open-source implementation (Netflix-Skunkworks/oci-agent) using a lightweight causal ML notebook with EconML. Implements the Principal-Actor-Critic workflow with the four design diagnostics, Crump-style trimming, and sensitivity analysis.
- **#concept Target Trial Emulation**: Netflix's pre-existing OCI philosophy: for any causal question, first ask what the ideal A/B test would be—even if infeasible. This clarifies the assumptions needed for credible answers. The agentic workflow inherits this philosophy and makes it operational.

---

## Critical Analysis

**The scaffolding-vs-one-shot result is the article's lasting contribution.** 9/10 vs. "not at all correlated with ground truth" on the same model is not a marginal improvement—it's a phase change. This should kill the "just wait for a smarter model" argument dead for any domain requiring methodological rigor. If Sonnet 4.6 can't do causal inference without a harness, GPT-6 won't either unless it internalizes the entire causal inference methodology—and even then, you'd want the harness for auditability.

**The Principal-Actor-Critic pattern is under-described.** The article tells us what each persona does but not how they're implemented. Is the Critic a separate LLM call with a different system prompt? Same call with a role-switch instruction? Are Actor and Critic the same model? These implementation details matter for anyone trying to replicate the pattern. The open-source repo presumably answers these questions, but the article should have been more explicit.

**The ACIC competition evaluation is honest but limited.** The authors acknowledge that synthetic datasets don't pressure-test semantic understanding—the LLM isn't being asked to reason about whether "days of engagement with Type X" is a valid treatment variable or whether the confounders make sense in context. This is the same limitation [[FrontierCode]] addresses for coding benchmarks: synthetic evals measure one thing well (statistical methodology) but miss the harder judgment calls that make or break real-world causal inference.

**The case study is the article's most persuasive section and it's only two paragraphs.** Early adopter bias → failed overlap diagnostic → Crump-style trimming → estimate drops to 25% of baseline. This is a complete, honest story of an agent catching a real methodological error that a human + baseline LLM missed. More case studies, fewer benchmark tables, would have made the article stronger.

**The "human-augmenting" framing is more than branding.** Most AI-in-science papers frame the AI as replacing or exceeding human capability. Netflix frames it as reducing toil so humans can focus on judgment. This is the right instinct for causal inference specifically (where assumptions are everything and can't be automated) and probably for most scientific applications of AI generally. The [[Man-Computer Symbiosis]] lineage is more productive than the automation lineage.

**What's missing: the failure modes of the scaffold itself.** The article shows the scaffold catching baseline errors but doesn't discuss when the scaffold fails—when the Critic misses a blind spot, when the diagnostics give false confidence, when the Principal trusts the credibility score too much. Every guardrail creates new failure modes (the [[Load-Bearing Assumptions]] problem), and a production system at Netflix scale has almost certainly encountered them. A frank discussion of those would be more valuable than the ACIC benchmark tables.

---

*Sources: [[summary/causal-inference-agentic-workflow]]*
*Last updated: 2026-06-22*
