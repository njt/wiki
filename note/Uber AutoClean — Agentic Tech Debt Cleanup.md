# Uber AutoClean — Agentic Tech Debt Cleanup

Uber's AutoClean autonomously deletes stale feature flags from production code — inlining the rolled-out value, propagating simplifications through call sites and signatures, fixing what CI breaks, and validating the diff before a human ever sees it. It is a mature production showcase of the "deterministic where possible, agentic where necessary" discipline: over 16,000 flags and a million lines of code deleted.

---

AutoClean targets a debt species everyone recognizes: once a feature flag is fully rolled out, its scaffolding quietly survives forever, because removing it competes with new product work. The essay's sharpest observation is that this debt now taxes agents too — they burn tokens parsing dead code every session, making cleanup self-reinforcing infrastructure for an agentic codebase.

The architecture is three loops. First, an agent with full repo access but a deliberately narrow toolset performs the rewrite, delegating boolean simplification to PolyglotPiranha (a deterministic code transformation engine) over MCP. Second, CI is the source of truth, and failures route through a fixer cascade: cheap deterministic fixers first, agentic fixers for judgment calls like "delete this test or rewrite it?" — each tightly scoped (a test fixer can only touch test files). Third, validation beyond green CI: static checks, then CodeCourt, a panel of single-property reviewer agents run in parallel whose verdicts aggregate to ship-or-abandon.

## Key quotes

> AutoClean is an agentic system, but that doesn't mean the agent should handle every step. We want to keep as much determinism in the loop as possible, especially for transformations that can silently change program behavior if they're wrong.

The central design rule, stated plainly. A boolean simplification error changes which branch executes in production — exactly the class of mistake LLMs are worst at and exact tools are best at. Compare [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]], which argues the same tiering as a general principle; AutoClean is that principle with production numbers attached.

> Without that limit, an agent will sometimes make a failing test pass by changing the production code underneath it, which is exactly the outcome we're trying to prevent.

Scope-as-guardrail: the fix that keeps agents honest here isn't a better prompt, it's a permission boundary. This connects to the whole sandboxing tradition — [[A Deep Dive on Agent Sandboxes]], [[Fences, not Sandboxes]] — but applied inside the pipeline rather than at the runtime boundary.

> Early on, we tried having a single agent review each diff end-to-end, but found that using focused reviewer sub-agent yielded better results.

Property decomposition over holistic review — the same convergence Cloudflare reports in [[Orchestrating AI Code Review at Scale]] (7 specialist agents behind a coordinator). One reviewer asked to "check everything" checks nothing well; one property per agent, run in parallel, aggregates into a decision.

> The human reviewer can focus on business-level judgment, like "should we remove this flag now?" rather than scanning for mechanical correctness.

This is the sharpest articulation of what AI review should free humans for — and it quietly separates two questions current review tools conflate. CodeCourt owns "is the diff mechanically correct?"; the human owns "is this change wanted?" That split is exactly the verdict/evidence layering Addy Osmani argues for in [[Own the Outer Loop]].

## Themes

#concept — deterministic core, agentic shell; scope-as-guardrail; property-decomposed review
#tool — PolyglotPiranha (Uber's code transformation engine) exposed over MCP
#pattern — fixer cascades (deterministic first, escalate to agents); ship-or-abandon gates rather than fix-and-warn

## Analysis

The most interesting thing about AutoClean is what it refuses to let the agent do. The naive version of this system — "LLM finds stale flag and deletes it" — is trivially buildable and quietly terrifying, because dead-code removal is semantically loaded: deleting a test that "covered" a now-dead branch can also delete the last guard on live behavior. Uber's answer is not to trust the model more but to shrink its blast radius three separate ways: deterministic tools for behavior-touching rewrites, file-scoped fixers, and property-checking reviewers who can only veto.

CodeCourt deserves attention as a review design. Its properties (no additions, preserved semantics, no intent drift, no agent artifacts) are all *negative* constraints — things a cleanup must not do — which are precisely what CI cannot check and what a single generalist reviewer blends together. The failure mode they're engineering against is the agent that passes CI by gaming it: deleting the failing test, or stopping early and leaving half-propagated dead code. Both failures are invisible to green builds, which is the recurring lesson of verification-first essays like [[Agentic Verification]] and [[Agentic Code Review]]: green is not a verdict.

Skeptically: this is a vendor post about a narrow, unusually well-instrumented domain. Feature-flag cleanup has unusually clean ground truth — the flag value is known, the transformation is well-defined, the properties are enumerable. Whether the fixer-cascade-plus-property-court shape transfers to messier refactoring (where "correct" is a judgment call, not an invariant) is asserted nowhere and remains the open question. But as a data point it's strong: a million deleted lines is the largest published number I know for autonomous deletion rather than generation, and it inverts the usual framing — the highest-value agent work here produced *negative* diff volume.

## Related

- [[Uber — Agentic Engineering Shift]] — this source is the concrete artifact behind Uber's broader agentic-platform story; it strengthens that page's platform narrative with a single system specified in detail.
- [[Orchestrating AI Code Review at Scale]] — complicates and confirms it: both independently converged on parallel single-property reviewer agents over generalist review, at different companies for different problems.
- [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] — AutoClean is the production embodiment of that tiering rule, adding the missing piece: *scoped permissions* as the mechanism that enforces the tiering.
- [[The Economic Benefit of Refactoring]] — complementary economics: Fowler measured refactoring making agent work cheaper; AutoClean shows deletion (the extreme refactor) being done by agents at scale, though with heavy scaffolding around each deletion.

---
*Sources: [[raw/ai-code-cleanup]], [[summary/ai-code-cleanup]]*
*Last updated: 2026-10-21*
