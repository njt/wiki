# Your .md File Is Not a Specification

The FizzBee team's tutorial-turned-argument that requirements written in prose — even carefully structured EARS prose — are systematically incomplete, and that a formal, executable specification is the cheapest way to find the gaps before the code exists. Through a deliberately tiny salon-booking example, it shows a model checker surfacing three distinct underspecified behaviors in minutes, each one forcing a product decision rather than a debugging session.

---

The setup is a jab at the spec-driven-development moment: "if we give them to a software engineer or a coding agent, will it build what we actually want?" The example is two sentences a PM might write for a salon booking app. EARS notation tidies them into three requirements — and the post's first real point lands immediately: R3 ("the system shall ensure all appointments are within the stylist's schedule") is an *invariant*, not a behavior, and invariants cannot be tested, only reasoned about from pre- and postconditions.

> "Invariants cannot be tested. You can only test for preconditions and postconditions. Invariants must be reasoned about from these testable pre- and post-conditions."

This is the load-bearing distinction of the whole piece, and it's one most spec-driven-development discourse skips entirely. Most SDD material treats "the spec" as a better-worded requirements doc; this post insists the spec must describe *actions and how they change state*, which is what Dynamic Logic's [A]p (necessarily-after) and <A>p (possibly-after) modalities are for.

The worked model is tiny — one stylist, one customer, one slot — and that's the point. The FizzBee model checker finds:

1. **The obvious gap**: `BookAppointment` has no precondition, so a customer books with no schedule at all. Fix: `require scheduled_to_work`, plus a new R2b for the rejection case.
2. **The subtle gap**: the stylist sets 9–5, a customer books 9am, the stylist changes to 10–6 — the old booking violates the invariant. This one is *not* a bug; it's a product decision with three defensible answers (block schedule changes, cascade-cancel, or flag for a manager). Formalization doesn't decide it; it surfaces it while it's still cheap.
3. **The gap the invariant itself missed**: with two customers, Customer#1 can silently overwrite Customer#0's booking. The post is candid that this survived the formal assertion — "formal methods don't automatically find every bug. The results are only as good as the properties and assertions we specify."

That honesty is rarer than it should be. Formal-methods advocacy usually sells completeness; this post sells *gap-finding with an explicit blindness to what you forgot to assert*, then shows the fix (interactive exploration of the state graph caught the overwrite) as a partial compensation.

The back half extends the payoff: roles model *who* performs each action (Customer, Stylist, System), the same spec serves as a stakeholder-exploritable state diagram, and model-based testing checks the implementation one transition at a time — refinement plus the "rejects must be spec-disabled" rule, which together amount to bisimulation for deterministic specs. Two details are sharp: don't re-assert the invariant on the implementation (redundant *and* weaker than transition conformance — block-on-conflict and cascade-cancel both preserve it), and "an implementation that rejects everything would pass the first rule alone."

The agent angle is direct: for real applications, coding agents translate requirements into FizzBee and refine the model as requirements evolve, with official skills for Claude Code, Cursor and Gemini CLI. Formal specification here is positioned as the verification layer specification-driven development currently lacks.

## Key themes

#concept — invariants vs behaviors; Dynamic Logic; refinement and bisimulation
#tool — FizzBee, model checking, model-based testing
#pattern — executable specification as the artifact both humans and machines reason about

## Opinionated take

This is the most concrete thing in the wiki on *what a specification actually has to contain* to be checkable. The wiki's SDD corpus (Spec Kit, SDDW, the SDD case studies) is overwhelmingly about process: templates, gates, traceability chains. Almost none of it asks the question this post asks — is the spec *well-formed enough to be wrong about something?* The salon example is doing real work: a three-requirements system with a genuine product decision hiding in it, found by a model checker in seconds, is a better argument for formal specs than any essay about rigor.

The weak point is the usual one for formal-methods evangelism: the example is toy-sized, and the post admits the real workflow is "agents write the spec." That shifts the trust question rather than answering it — you now need to verify the spec faithfully captures the requirements, which is its own review problem. The post's honest caveat about assertion blindness cuts both ways: if the model missed the overwrite bug until a human poked at a diagram, a spec written by an agent on your behalf will miss whatever *you* didn't think to ask about either. Formal analysis raises the floor of specification quality; it doesn't remove the human judgment about what matters — the same relocation of judgment the rest of this wiki keeps documenting.

The other quiet contribution is the EARS critique. EARS is the default "good enough" requirements notation, and the post's point that EARS happily encodes untestable invariants is a genuine gap in its adoption story.

## Related pages

- [[The Coming Need for Formal Specification]] — the argument-level case this post is the tutorial for; the salon example gives that argument its first worked demonstration, invariant-not-testable distinction included.
- [[Bend — A Proof-Gated Language for AI-Written Code]] — a different point on the same spectrum: Bend gates *code* with proofs, FizzBee gates *requirements* with model checking; both answer "how does the machine check the thing instead of generate it."
- [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] — the cascade-cancel vs block-on-conflict fork is exactly the kind of silent resolution bl00cyb warns about; here the model checker surfaces it as an explicit decision instead of letting an agent pick a default.
- [[TDD Inside the Agent Loop — Theater or Actual Value?]] — a useful tension: Thoughtworks found prescribed test-first loops added cost without quality, while this post's model-based testing derives tests from the spec mechanically; both are verification-first, but only one has a formal oracle to test against.

---
*Sources: [[raw/formal-analysis-in-requirements-specification]], [[summary/formal-analysis-in-requirements-specification]]*
*Last updated: 2026-10-09*
