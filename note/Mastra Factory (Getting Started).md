# Mastra Factory (Getting Started)

Mastra Factory is an open-source software factory that turns GitHub issues into plans, implementations, and reviewed pull requests through a configurable pipeline of stages, skills, and approval gates — self-hostable or run on Mastra platform. This page is its getting-started guide, which doubles as a precise statement of the product's theory of human oversight.

---

The scaffold step already encodes a philosophy: the Factory project lives in its own directory, separate from the repository agents change, with independent Git histories. The factory is infrastructure about your codebase, not inside it.

The walkthrough is a staged demo with a deliberately boring issue (add a Contributing section to a README), and its guardrails are the real content:

- **Three distinct human gates.** Acceptance authorises a task to enter work; plan approval authorises the approach; the merge happens through the repo's normal review process. "It's separate from approving the plan and from merging the eventual PR."
- **Review is a separate session from authoring.** The PR's card on the Review board has its own review session, so the agent reviewing is not the agent that wrote the code.
- **Verify the verifier.** "An agent saying a check passed is useful context. Also inspect the tool output and the PR's checks." This is the docs quietly admitting that agent self-reports are not ground truth.
- **No post-merge reassessment.** "Check the acceptance criteria before merging, because Factory doesn't reassess the task after the merge." The human owns the criteria, permanently.

## Opinionated take

This is a well-designed factory harness in the lineage the wiki has been tracking, and it gets the details right: separate review sessions, explicit plan approval, credential encryption as a first-class setup step, and an insistence on inspecting tool output rather than trusting agent claims. It is also, notably, a *productised* factory — the opposite pole from the dot-file dark factories ([[The Dark Factory is a DOT File]]) and hand-rolled kanban harnesses ([[Managing Agents via Kanban Boards]]). Mastra is betting teams will buy a factory as a supported platform with auth, storage, and sandboxes swappable independently, rather than assemble one from skills and scripts.

What the guide doesn't show is the hard part every factory source in this wiki eventually hits: what happens at volume, when the three human gates become the bottleneck, and what "stages, skills, and approvals your team sets up" looks like after six months of configuration drift. The walkthrough is one README issue; [[What Matters After the Software Factory Works]] is the question this page doesn't ask.

## Key quotes

> Factory turns issues into plans, implementations, and reviewed pull requests, following the stages, skills, and approvals your team sets up.

The product definition, and a telling one: the pipeline shape is fixed, the *configuration* is yours. Factories as platforms sell you the loop and let you tune the gates.

> Acceptance authorizes the task to enter work. It's separate from approving the plan and from merging the eventual PR.

Three gates, three different objects of judgment: is this worth doing, is this the right way, and is the result correct. Most lighter-weight harnesses collapse these into one approve button.

> An agent saying a check passed is useful context. Also inspect the tool output and the PR's checks.

The docs' own verification doctrine — self-report is evidence, not proof. Aligns squarely with the wiki's verification-first sources.

## Related pages

- Strengthens [[Software Factories, Light and Dark]]: this is a concrete light-factory product, with the per-stage human switches (accept, approve, merge) Osmani's synthesis describes, shipped as installable software rather than argued as an essay.
- Nuances [[Managing Agents via Kanban Boards]]: Factory's Work board with Intake/Investigate/Review cards is the kanban-for-agents pattern formalised into a product, with the crucial addition of a separate review session per PR.
- Complicates [[Crawl, Walk, Run to the Software Factory]]: the guide's walkthrough is entirely "walk" — every gate held open for the human — which raises the question of what "run" looks like on a platform whose defaults are built around approval gates.
- Related: [[Droid 1.0 — Building a Frontier Harness (Factory)]] for another vendor's harness; [[Inside a Software Factory]] for a practitioner's account of running one.

---
*Sources: [[raw/factory-mastra-ai]], [[summary/factory-mastra-ai]]*
*Last updated: 2026-11-20*
