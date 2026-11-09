# Beyond Code Generation: Agentic Engineering on GitHub

GitHub's own guide to what "agentic engineering" means in practice: a Plan → Build → Verify → Observe loop where agents are deployed at every stage of the lifecycle, not just code generation. Built on one striking pair of statistics — 1.2 billion commits in a single week (400%+ year-on-year growth) against a 441% increase in PR review time at high AI adoption — it is, at heart, an argument that the bottleneck has moved from writing to reviewing.

---

## What it argues

The article opens by puncturing the productivity narrative: yes, commit volume exploded, but "do you feel 400% more productive than you were last year?" Generating code got cheap; gathering requirements, steering implementation, reviewing changes, and investigating production did not. The remedy offered is a full-lifecycle workflow, with each stage anchored to specific Copilot capabilities (Plan Mode, cloud sandboxes, Copilot Dreaming, stacked PRs, Agentic Autofix, Agent Merge, observability connectors).

The strongest section is Verify, because it names a concrete failure mode most teams will recognise: the 47-file "small refactor" PR where the one piece of shared validation logic you actually wanted reviewed is buried under formatting churn and hundreds of generated test lines. The prescriptions are structural, not motivational — smaller stacked PRs, automated quality checks before a human opens the diff, review depth matched to change complexity.

## Key quotes

> "Chances are, you're generating more code than you were a year ago. But ask yourself: do you feel 400% more productive than you were last year?"

The rhetorical move that justifies the whole piece. Throughput metrics are leading indicators of a queue forming downstream — and the queue is review.

> "a Faros AI study found that pull requests spent 441% longer in review when teams went from low to high AI adoption."

The same 441% figure already circulating in [[Agentic Code Review]] — here it's the vendor's own framing device, which is telling: even GitHub has to sell its product by admitting AI made review worse before offering to fix it.

> "You open a pull request labeled 'small refactor.' It changes 47 files. You wanted feedback on shared validation logic, but it's buried among updates to every endpoint that calls it, formatting changes, and several hundred lines of generated tests."

The most honest paragraph in the piece. It describes the exact failure mode of unreviewable agent-generated diffs — and notably, the fixes proposed (stacked PRs, pre-review quality gates) are process changes, not model improvements.

> "Writing code is more than shooting off a single prompt to your LLM and seeing what you get back."

A vendor gently talking its own users out of the one-shot mental model — consistent with the craft-level arguments elsewhere in the wiki.

## Key themes

#concept #workflow #tool #pattern

## Opinionated take

This is a product catalogue wearing a workflow essay's clothes, and it should be read that way — every stage resolves to a named GitHub SKU. But the catalogue is unusually well-organised around a real claim, and the claim is the interesting part: the industry's most prominent code-hosting platform is publicly telling customers that AI adoption made review the bottleneck and that the answer is process redesign (smaller diffs, automated pre-review gates, agent-handled follow-ups) rather than smarter models. The 441% figure appearing here independently corroborates what [[Agentic Code Review]] reports.

Two things deserve scepticism. First, the Observe section is thin — "connect your observability tools via connectors" is not a workflow, and incident investigation is exactly where agent errors are most expensive. Second, "Agent Merge" and conditional auto-merge quietly assume the verification stack is trustworthy; if review quality is already strained, delegating the merge decision to the same pipeline is the riskiest step in the loop, presented as the easiest. The article is best read as a well-informed checklist of what a mature agentic org needs — with the vendor's optimism about autonomy taken at half price.

## How it connects

- [[Agentic Code Review]] — cites the same 441% Faros figure; this source is the vendor-side confirmation that the review bottleneck is now mainstream enough to structure an entire product pitch around.
- [[The End of Code Review]] — complicates Monperrus's argument: where he says mandatory human review is indefensible, GitHub's answer is to keep human review but shrink and focus it (stacked PRs, depth-matched automated review) — a middle path between abolition and status quo.
- [[Agent Coding Workflow]] — a canonical Plan/Build/Verify/Observe articulation of the day-to-day loop, strengthening the planning-ritual and verification-over-generation strands of the topic page.
- [[Review AI Coding-Agent Work Without Losing the Next Step]] — the alterac handoff checklist operates at the level of one completed agent run; this source zooms out to the whole lifecycle, showing where those handoffs sit inside Plan → Build → Verify → Observe.

---
*Sources: [[raw/beyond-code-generation-a-developers-guide-to-agentic-engineering]], [[summary/beyond-code-generation-a-developers-guide-to-agentic-engineering]]*
*Last updated: 2026-11-09*
