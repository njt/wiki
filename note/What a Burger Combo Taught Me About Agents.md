# What a Burger Combo Taught Me About Agents

kinfey's Microsoft blog post is a build-log of a small demo: order a burger combo through two parallel agent stacks, one a conventional Copilot agent loop, one a Jev-fronted harness, and report honestly what the division of labor buys. Its real thesis is architectural, not product: let the LLM handle open-ended work, let Jev (TypeSafe's System One judgment model) handle bounded decisions, let plain code hold permissions and workflow boundaries.

---

## What it argues

- The LLM/chef vs agent/restaurant distinction: the model supplies language understanding and reasoning; the harness supplies context, tool access, execution, and stopping conditions. "The problem is not that LLMs are bad at their job. It is that we sometimes give one execution mechanism every job in the building."
- Jev is an order form, not an essay: it accepts state and predefined questions and returns typed decisions (Choice / Noul / Score) with probabilities. Six questions go in one `system_one` request instead of six conversational turns.
- Jev ≠ model router: a router picks *which model handles the request*; Jev decides *what the request means and which business branch applies*. Both paths kept GPT-6-astra — nothing was swapped for a faster model.
- The pipeline: `request → Jev bounded decisions → local Python gate (thresholds, required fields, permissions, tool filtering) → Copilot SDK agent loop → GPT-6-astra + MCP`.
- Honest measurement: end-to-end time fell 113.28s → 35.41s (~68.7%) and input tokens 251K → 126K, but the author repeatedly refuses the causal claim — the optimizations are confounded, and an equally lean non-Jev control is needed to isolate Jev's contribution.

## Key quotes

> "The problem is not that LLMs are bad at their job. It is that we sometimes give one execution mechanism every job in the building."

The cleanest statement of the allocation-of-responsibility thesis. It reframes agent inefficiency as an architecture decision rather than a model-quality complaint.

> "By the stopwatch, it won. By the user's goal, it failed."

The fastest early run truncated the menu and had no tool to read the full output — so it never computed the price. A benchmark-shaped cautionary tale about optimizing latency without a success criterion that checks the actual goal.

> "Selecting only from the menu does not guarantee selecting what the customer meant."

On TypeSafe's "no hallucinations" framing: schema conformance, semantic correctness, and authorization are different properties. This is the right counterweight to marketing language, and it applies to any structured-output story.

> "A better agent is not necessarily an agent with more autonomy. Sometimes it is a system that has finally learned to divide the work."

The closing lesson, and the inverse of the usual capability-maximizing pitch.

## Themes

#concept — division of labor between generative models, typed decision models, and deterministic code
#pattern — classifier-gated agent loop with a Python permission/tool-filtering gate
#tool — TypeSafe Jev, GitHub Copilot Python SDK, McDonald's China MCP
#concept — calibrated uncertainty as an API surface (probability distribution vs confidence as separate things)

## Opinionated take

This is the most methodologically humble vendor-adjacent demo write-up you'll see: the author pre-empts his own success story, labels the 0.75 confidence threshold a demo setting rather than a safety standard, and tells you the 68.7% figure is confounded. That honesty is the post's best feature, because the underlying claim — Jev as a cheap, typed pre-processor that shrinks what the frontier model must see — is genuinely interesting and badly needs clean measurement to prove. The demo itself can't provide it.

The most instructive failure was not the benchmark confound but the day-one design mistake: passing only the six structured fields to the executor, which then couldn't search for a restaurant. The fix — retain the raw request as context alongside the typed plan — is the whole lesson of hybrid architectures in miniature: typed decisions are useful, not a substitute for unstructured context. The other useful caveat is that the "Jev Harness" still runs a full agent loop downstream; it's a hybrid prototype, not a workflow engine, so the neat three-way division is aspirational at the edges.

---

This source strengthens [[System One Models and Jev]] with a second, independent implementation of the Choice-primitive pattern in a real codebase, including the practical lesson about what structured decisions must not replace. It nuances [[Model Routing Is Simple Until It Isn't]] by sharpening the router-vs-classifier distinction: routing picks the model, Jev picks the business branch — two orthogonal axes that source treats separately. It complicates [[Jev Can't Be Calibrated]] with a more optimistic but caveated data point: the author treats calibrated uncertainty as genuinely useful for gating, while still warning that confidence is not correctness. And it gives [[Building with Jev Skill]]'s skill-based view of the same primitives a concrete harness-scale counterexample — parallel batched questions, one shared state, thresholds enforced in code rather than prose.

---
*Sources: [[raw/4558830]], [[summary/4558830]]*
*Last updated: 2026-10-03*
