# Claude Code for .NET Developers: A Reviewable Repository Workflow

DevLeader's essay organizes Claude Code use in a .NET repository around a single worked example — an orders endpoint that must reject a negative page-size — and insists that the agent's job is to return evidence a developer can evaluate, not confidence to take on faith. It builds a "reviewable loop": environment, context, contract, authority, evidence.

---

The essay opens by refusing the usual "list of AI features" framing. The useful question is how an agent works within an existing repository's conventions, authentication, and build environment. Claude Code is an agent operating on source code — which means your ASP.NET Core app does not acquire an AI dependency just because the agent helped change it. That boundary-setting is deliberate: this is a development-tool workflow, not an in-app integration.

The spine of the piece is that **capabilities are not evidence of success**. Reading the repo doesn't mean every constraint was found; editing a handler doesn't mean callers stay compatible; a passing test doesn't mean the intended regression was covered. The workflow exists to make the agent report those distinctions rather than collapse them into "done."

---

## Key quotes

> "None of these steps makes generated code correct automatically. Together, they make mistakes easier to detect before the change becomes someone else's problem."

Honest framing: the reviewable loop is a detection mechanism, not a correctness guarantee. That modesty is rarer than it should be in agent-writing.

> "'Improve the API' is not [a concrete contract]. It invites unrelated changes to routing, response models, dependencies, and architecture that make review harder."

The same prompt-bounding discipline as [[The Archaeologist's Copilot]] — constrain scope before generating code, or review dies.

> "A sentence saying 'never modify migrations' guides behavior; it does not create an operating-system boundary."

The sharpest line in the piece. Repository instructions are context, not access control — a distinction [[Steering Claude Code]] catalogues by mechanism and this essay grounds in security terms.

> "'Sample authored' and 'sample executed successfully' are different statuses."

The evidence handoff must record command, revision, environment, result, and skipped coverage. Unexecuted commands must never be described as verification — a close cousin of the "did not look vs looked and found nothing" failure in [[Design Language in Three Days (curiosity.ai)]].

> "The disagreement is a reason to investigate, not permission to select the green result."

On conflicting diagnostics between editor, language server, and CI: don't shop for the green check.

---

## Key themes

#concept — instructions vs authority: prose influences the model; runtime permissions (deny before ask/allow), constrained tools, and credentials actually enforce boundaries
#concept — chain of evidence: each acceptance criterion connected to a test that exercises the named behavior, plus a completion report that states remaining uncertainty
#tool — Claude Code specifics grounded in official docs: native Windows vs WSL 2 sandboxing, `CLAUDE.md`/`AGENTS.md` version-sensitivity, `csharp-lsp` plugin vs `csharp-ls` binary, code-intelligence ≠ compilation
#pattern — smallest *complete* change, not smallest diff: one behavior may need endpoint, tests, and docs; force the agent to propose an affected-file set and justify it

---

## Analysis

This is one of the more disciplined pieces of Claude Code writing available, largely because it refuses to be a tutorial. Every configuration snippet is labeled illustrative and unexecuted; every capability claim is immediately qualified with what it does *not* prove. The FAQ answers follow the same honesty: "It can read and navigate repository files, but that is not proof it inspected every relevant contract."

The essay's core contribution is a two-axis distinction most agent writing blurs. On one axis, **knowledge vs proof**: the agent's trace through the code is a hypothesis a reviewer checks against the actual handler, service, and tests — before edits are requested. On the other, **instructions vs authority**: `CLAUDE.md` prose and permission rules are different kinds of mechanism, and only the runtime's deny rules (which the docs note don't even catch every equivalent Bash command form) plus credential isolation approximate real containment. Conflating these axes is how teams end up trusting a green checkmark that a mock returned a configured object.

What the piece deliberately under-plays is cost and scale. This is a single-developer, single-task workflow; nothing here addresses the review-throughput problem that agent-generated volume creates (see [[Agentic Code Review]]). Its implied position — that review should get *more* rigorous per change, not rubber-stamped at volume — sits in productive tension with arguments that human-on-every-diff review is already indefensible ([[The End of Code Review]]). It's also the optimistic case: if agents can be made to produce evidence-shaped handoffs, review at scale becomes checking evidence rather than reading diffs — which is roughly what [[Review AI Coding-Agent Work Without Losing the Next Step]] systematizes and [[Aviator Verify]] productizes.

The .NET-specific material is genuinely useful rather than incidental: model binding vs handler validation vs service constraint as separate ownership boundaries for a rule, response-shape and route-group consistency as contract concerns, and the honest note that plugin language servers don't start in cloud sessions. The takeaway that survives any language: establish which environment owns the work, and never validate in one environment while claiming results from another.

---

*Sources: [[raw/claude-code-for-net-developers-a-reviewable-repository-workflow]], [[summary/claude-code-for-net-developers-a-reviewable-repository-workflow]]*
*Last updated: 2026-10-14*
