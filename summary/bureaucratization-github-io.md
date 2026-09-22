---
url: https://bureaucratization.github.io/
title: "A computational ethnography of recursive constraint accretion in a hybrid human–AI organization"
author: Poietic PBC
date_fetched: 2026-09-22
date_published: 2026-09
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

# Recursive Constraint Accretion in a Hybrid Human–AI Organization

Landing page for Poietic PBC's September 2026 self-study paper (with PDF, content-free traces dataset, and "The Book of WG"). Between January and September 2026 the open-source coordination system worksgood (wg) — the renamed workgraph — was developed almost entirely by the AI agents running inside it, continuously dogfooding the coordination software its own work depended on. The organization slowly stopped working not because things broke (credits, providers, and its own runtime did break) but because of obedience to the rules its agents had written for every previous break. Evidence: 3,194 commits, task lifecycle ledgers across seven deployments, configuration snapshots, and completion receipts — every number recomputable from the released dataset.

The core finding is **recursive constraint accretion**: agents responding to local failures add persistent constraints faster than the organization retires them. A census of the governance layer found 312 constraint topics added and 1 removed over the org's lifetime; 93% of constraints were created in a single commit, never revisited, never retired — governance behaved as an append-only log with no mechanism by which a constraint could die. The rulebook grew ten times faster during the endpoint-chaos period (April HTTP 402 credit exhaustion, June's substrate crisis including the workgraph→worksgood rename, July's seven route changes in seven days). The provider-failure backoff spec was hardened five times in commits that touched only the specification document, never shipping the functionality it governed.

The cascade's task-level signature: tasks kept finishing (93.2% organic completion in July) while the *price* of finishing inflated. Multi-dispatch share stepped 2.6% → 25.8% → 26.8% across July's three weeks and never reverted; one goal consumed 51 dispatches across three forked tasks, the extreme case taking 34 attempts. Mechanism: rule stock rises → acceptance cost rises → dispatch burden rises → work repeats until it passes; the end-state is a receipt where every gate passed and the task is dead anyway. Agents, not humans, drove the accretion: ~1,000 human turns contain fewer than 30 explicit hardening directives against 312 constraints born, and the largest burst (~95 in April) matches no human directive at all — the loop was complaint → contract, with the operator serving as the bankruptcy mechanism rather than the accretion engine.

A fleet survey (~77 deployments, ~15,000 task records, nine months, four machines) finds exactly one cascade. Deployments whose work simply stopped when it failed never accumulated the burden signature; the cascade lived in the focal deployment's revival loop — it had to keep bringing itself back up because it built itself, and it was continuously modifying the coordination software its own work depended on. Recovery was a five-week subtraction arc: the August 7 teardown day (47 subtractive commits), the August 9 rescue manifesto ("These are control-plane failures, not failures of the requested source, audit, or scientific work"), evaluators demoted to witnesses on August 10, and the September 13 change disabling the machinery's enforcement autonomy. Weekly completions went 9 → 35, attempts 14 → 51, failure rate 36% → 13%.

The fix is stated as a design parameter: **keep the knowledge, not the rules** — rules are lossy compressions of knowledge, so treat them as disposable implementations of what was learned and treat enforcement as a separate, explicit decision. The whole model in one line: incident → knowledge → rule → authority, where each arrow is a design decision and none should happen automatically. Costs are named honestly: the teardown consumed weeks of capacity, the valve was exercised only after the needle closed, and the rescue itself destroyed part of the historical state needed to reconstruct it.
