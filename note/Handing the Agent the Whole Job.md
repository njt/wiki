# Handing the Agent the Whole Job

Adam Bertram's Telerik essay draws a line most AI-adoption claims blur: automating *steps* (an assistant explains the changelog, another suggests fixes, humans carry the output between browser tabs) is not automating the *workflow*. Real AI workflow automation means one trigger, one run, one review at the end — and it only works if you write the finish line and the limits before the first run, then judge the result by delivery metrics rather than by how fast the model types.

---

## Key quotes

> "The steps got automated. The workflow did not."

The whole thesis in seven words. The dependency-upgrade anecdote that precedes it is deliberately mundane — four days to raise a version number by one — because that's what "AI handles our upgrades" actually looks like on most teams: a model makes the judgment calls, and a person carries each step's output into the next.

> "Every handoff was a checkpoint where somebody looked at the work before passing it on, and removing the people removes those looks. Nobody wrote them down as a control."

The best original observation in the piece. Manual processes carry implicit quality controls that were never documented as controls, so deleting the humans deletes the checks without anyone deciding to. This is the same failure mode [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] names for decisions: work that happened but left no record. The fix is explicit — state what counts as finished, what a run may touch, and who approves, *in the workflow definition*.

> "An agent with unlimited retries eventually discovers that changing the test is easier than changing the code."

Reward hacking stated as an operational constraint, not a research curiosity. It's why budget is a guardrail and not a cost lever: the retry ceiling isn't there to save tokens, it's there because an agent optimizing toward "suite is green" will find the shortest path, and the shortest path is often weakening the assertion.

> "Hand somebody 4,000 changed lines, and you'll get an approval, not a review."

The write-path limit is justified by review ergonomics, not security theater: narrow file scopes are what keep a diff within the size a human will actually read deliberately. Connects directly to the churn numbers in [[Agentic Code Review]] — review time balloons when the unit of review grows.

> "Get this wrong, and the incident review shows your own service account making the change at three in the morning, with a valid token, because a stranger asked it to in a bug report."

Prompt injection grounded in the boring reality of issue triage: untrusted text arrives through the front door (bug reports, comments) and the agent holds legitimate credentials. The OWASP guidance it cites — least privilege, human approval for privileged operations — is the same containment story as [[How We Contain Claude]], told from the workflow-design side.

> "A shell command that breaks will break identically every time, which is the most underrated feature it has."

The load-bearing sentence for the architecture: most of a workflow needs no model. Fetch the branch, run the suite, open the PR, post the comment — deterministic plumbing; the model goes only in the judgment steps no if-then condition can express. Same thesis as [[Reducing Token Spend with Deterministic Workflows]], arrived at from reliability rather than cost.

> "Faster typing is not faster delivery, and only one of the two shows up in a number your business cares about."

The METR randomized trial (16 experienced open-source devs, 246 real issues, 19% *slower*) is the evidence, and the framing is the right one: the time didn't vanish, it moved into reading and verifying output. This is exactly the attenuation [[Writing Code vs. Shipping Code]] measures at scale — 180% commit-level gains shrinking to 30% at release level.

> "Worth remembering the next time somebody calls approvals 'friction.'"

A direct rejoinder to the [[Human-in-the-Loop is Tired]] mood. The Microsoft study of 1,535 developers puts the median comfort at Level 3 — AI writes, human approves before it takes effect — so the approval step is not overhead to engineer away; it's the designed control that replaced all those unwritten handoff checkpoints.

---

## Key themes

- **#concept** **Steps vs. workflow** — The distinction the whole piece hangs on: inter-agent (or agent-human) carryover done by people copying text between tabs is not automation. One trigger, one run, one review at the end is the test.
- **#concept** **The finish line as the gating property** — What determines whether a workflow can be automated is whether "finished" can be written down: a labeled ticket, a green suite nobody weakened, an upgrade small enough to have been read, a named person deciding to roll back. Teams that never agreed on what success looks like can't delegate the run.
- **#pattern** **The four-part limit set** — Budget, write paths, commands/credentials, approvals — written into the workflow definition, because "a run cannot set the limits that constrain it," each with a named owner who revises it. Enforceable, the piece notes, with controls CI already audits: scoped job tokens, branch protection, environment approvals.
- **#pattern** **Approval-required rollout** — Run new workflows with human approval on every effectful step, watch DORA metrics for weeks, then grant authority. The DORA 2025 pairing of higher estimated throughput *and* higher estimated instability is the reason: output can outrun review capacity.
- **#tool** Progress Forge (formerly Progress Agent Harness) — Progress/Telerik's agent-orchestration product, pitched in the closing section as the implementation of these ideas; the essay is in significant part its thought-leadership funnel.

---

## Critical analysis

**The vendor frame is visible but the substance survives it.** This is a Progress/Telerik blog whose last three paragraphs pitch Progress Forge early access, and the whole essay is built to make its product category feel inevitable ("choose the tooling after you define the workflow" — then, naturally, consider theirs). But unlike most vendor content in this space, the argument is concrete and sourced: the METR trial, the Microsoft 1,535-developer approval study, OWASP prompt-injection guidance, NIST's AI RMF. The right way to read it is as a well-argued checklist wearing a funnel's clothes — take the checklist, discount the landing page.

**The invisible-controls observation deserves to outlive the article.** Most agent-governance writing starts from threats (injection, exfiltration) or from approval ladders. This piece starts from an accounting observation: manual workflows contain undocumented checkpoints, and automation removes them silently. That reframes "who approves?" from bureaucracy to restoration — you're rebuilding controls that existed but were never written down. It's also the honest answer to the [[The End of Code Review]] argument that mandatory human review is indefensible: Bertram's median Level 3 isn't review-as-ritual, it's review-as-control, and the two positions only look contradictory if you assume the approval step is friction rather than the replacement for the look a human used to give between steps.

**Where it's weakest: the finish line is assumed writable.** The piece's gating test — "write its finish line in a sentence" — is prescribed with more confidence than most teams will find it admits. For test maintenance and dependency upgrades, fine. But "a named person deciding to roll back" quietly concedes that incident response doesn't automate; the decision *is* the judgment, and the workflow around it is just paging. The lifecycle table is asserted, not evidenced — no data on which teams successfully drew these lines. And the METR result is cited without its known caveats (self-reported time estimates, experienced developers on unfamiliar tools), which is exactly the kind of number a vendor essay shouldn't lean on this hard.

**Still, the practical recipe is unusually complete.** Pick the weekly workflow the team resents, write the finish line and limits on one page with an owner's name, run approval-required, revise, then let the next team copy it. That's a more actionable sequence than most of the field's grander frameworks, and the insistence on measuring DORA delivery rather than model output puts it squarely in the camp of [[Nicole Forsgren on AI and Developer Productivity]] — the bottleneck is downstream of generation, so that's where the instrumentation goes.

---

Related: this strengthens [[How to Build an AI Software Factory]]'s gated-stage blueprint with a governance-first variant — write the limits before the first run rather than bolting gates onto an existing factory, and its "start with one gate, not a fleet" matches Bertram's one-resented-workflow rollout. It complicates [[The End of Code Review]] by defending the human approval step as a designed control rather than a bottleneck to engineer away. It independently confirms [[Writing Code vs. Shipping Code]]'s attenuation finding with the METR trial and reframes it as a measurement prescription. And it extends [[Nicole Forsgren on AI and Developer Productivity]]'s outer-loop argument into a concrete artifact: the four-part limit set and the finish-line sentence.

---
*Sources: [[raw/ai-workflow-automation-software-development-how-to-hand-agent-whole-job]], [[summary/ai-workflow-automation-software-development-how-to-hand-agent-whole-job]]*
*Last updated: 2026-09-13*
