# How agent.ceo Fixes Its Own Bugs

A compact Onomos case study: on one ordinary working day in June 2026, the agent organisation running the agent.ceo platform found, root-caused, fixed and verified four of its own bugs — and the article argues the interesting thing is not the fixes but the process rules that made agent ownership safe: a durable task registry, a checkable definition of done, mandatory regression tests plus a second reviewing agent, no silent failure paths, and access limits enforced by construction.

---

## The rules, not the fixes

The author is explicit that "agents wrote fixes" is the wrong headline. What matters is "the rules that let them own the step." The piece maps five rules onto a readiness checklist:

- **Work in a system, not a head** — tasks live in a registry with a durable record.
- **"Done" is externally checkable** — tests pass *and* someone other than the author verifies the behaviour.
- **Every fix carries regression tests** — and a second agent reviews; that day the review caught a second fault in the same area.
- **No silent failure paths** — a rule written *after* the day, once a failure taught them why.
- **Least privilege by construction** — after a test task leaked into the production registry, tests were redirected so they cannot touch production stores at all.

The most revealing anecdote is negative: an agent handed an unassigned urgent task **did not run it** — it traced where the work came from and flagged it. That is ownership behaving like accountability, not enthusiasm.

## Key quotes

> "What matters for an engineering leader is not that agents wrote fixes. It is the rules that let them own the step."

The inversion is the whole argument: autonomy is a consequence of process discipline, not a substitute for it. Agent-readiness and human-readiness turn out to be the same checklist.

> "None of these rules is specific to agents. Each is a written owner, a check that can fail, and a limit on access."

This is the piece's claim to generality — the same rules make the process better for human teams today, which is why "preparation pays off even before you delegate."

## Take

This is a vendor's self-report about its own product, so the "four bugs in one day" headline deserves scepticism — but the *rules* are checkable, and they are the right ones. The pattern here is the same one running through [[Inside a Software Factory]] and [[Agentic Verification]]: the unit of quality is not the smart model but the loop of durable task, independent verification, and bounded access. The refusal anecdote is the best part, because it shows the process constraining the agents *upward* — flagging unowned work rather than seizing it — which is far more evidence of real ownership than four green test suites.

It also quietly complicates the harness-first story in [[Your AI Agent Is Only as Good as the Harness Around It]]: agent.ceo's wins here came less from harness cleverness than from boring engineering governance — registries, reviewers, least privilege — applied to agents as if they were junior engineers. And the "no silent failure paths" rule, learned the hard way, echoes the failure-path discipline argued in [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and the sandboxing-by-construction stance of [[Fences, not Sandboxes]].

#concept #pattern

---
*Sources: [[raw/how-agent-ceo-fixes-its-own-bugs]], [[summary/how-agent-ceo-fixes-its-own-bugs]]*
*Last updated: 2026-11-09*
