---
url: https://onomos.ai/articles/how-agent-ceo-fixes-its-own-bugs
date_fetched: 2026-11-09
---

## One working day

agent.ceo is an autonomous development system built by our group. Its agents develop, test and operate the platform itself. On 10 June 2026 its agent organization found, root-caused, fixed and verified four platform bugs in one working day, during ordinary work, with no bug hunt scheduled. The full account is public.

What matters for an engineering leader is not that agents wrote fixes. It is the rules that let them own the step.

## The rules, against our checklist

| Checklist question | How agent.ceo answers it | 
|---|---|
| Is the input in a system? | Work arrives as tasks in a registry with a durable record, not in someone’s head. | 
| Is “done” defined so someone else can check it? | A fix is done when the tests pass and someone other than the author verifies the behavior. | 
| Can the result be checked by a test or a second reviewer? | Each fix ships with regression tests. Another agent reviews the change; that day, the review found a second fault in the same area. | 
| Is a mistake cheap to detect and undo? | The rule written down after that day: no silent failure paths. A step that fails says so. | 
| Is access limited to what the step needs? | After a test task leaked into the real registry, tests were redirected so they cannot touch production stores, by construction. | 

One more rule shows why this is ownership and not automation: an agent handed an urgent task nobody had assigned did not run it. It checked where the work came from and flagged it.

## What this means for your process

None of these rules is specific to agents. Each is a written owner, a check that can fail, and a limit on access. That is why preparation pays off even before you delegate: the same rules make the process better for your people today.

The readiness checklist walks one of your processes through the same questions.
