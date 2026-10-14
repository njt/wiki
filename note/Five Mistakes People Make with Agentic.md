# Five Mistakes People Make with Agentic

intellectronica's "Factory Life" essay names five habits that stop a practitioner moving from *directing* agents to *delegating outcomes*: micromanaging, defaulting to the most expensive model, failing to give the agent taste and a way to check its work, building a "puppet show" of personas instead of separating real functions, and treating prompts and skills as if they enforced anything. The framing device is the **software factory**: give an agent an outcome, context, and a way to check its work, and let it carry the task through asynchronously while you decide what matters.

---

## The argument

The pivot from chat to delegation is measured by a single question: *"could I hand over this task and come back an hour later to the result I meant?"* Everything else hangs off that. Two mistakes live on the delegation axis itself (micromanaging, over-specification), two on the resource axis (model choice, multi-agent sprawl), and one — the sharpest — on the enforcement axis: sentences in AGENTS.md are not infrastructure.

## Key quotes

> "I can ask an agent to do something, watch it produce good work, and still spend the next hour telling it what to do next."

The opening is a confession most practitioners will recognise: the agent is busy, but you are still the bottleneck. The essay's whole project is making the *human* step back, not the agent more capable.

> "I once put an 'always write tests' rule in my standing agent instructions. When asked to change a README, an agent tried to write a test for the README."

A perfect miniature of mistake three's problem: the rule was global, the agent had no way to subordinate it to context, and the result was enthusiastic nonsense. The lesson isn't "write better rules" but "check what happens when a rule meets a task it wasn't meant for."

> "There is no prize for employing the most imaginary executives."

The puppet-show diagnosis. Multi-agent splits need a *function* — parallelism, narrower permissions, a persistent monitor — not an org chart. The test: does the split actually help, or does it only add handoffs?

> "We have asked a sentence to enforce a boundary."

The load-bearing line of mistake five. A model *interprets* instructions; behaviour shifts with model version, context, and retrieved material. If the agent can skip your check or continue after it fails, the check is "optional in practice." Enforcement belongs to the system: file permissions, sandboxes, identities without publish rights, pre-action gates.

## Key themes

#concept — factory mode vs chat mode; delegation measured by the one-hour-away test
#pattern — give the agent a way to check its work; start with one agent and split on demonstrated need
#tool — AGENTS.md, skills, and example corpora as homes for context

## Critical take

What elevates this above the usual "five tips" listicle is mistake five, which quietly reframes the other four. Mistakes one through four are about *calibration* — how much direction, which model, how many agents — and can be tuned by habit-breaking. Mistake five is about a category error: mistaking probabilistic interpretation for deterministic enforcement. The essay's distinction between "did the work pass the check?" and "is the agent authorised to take the next action?" is the most architecturally useful sentence in the piece, and it's the one most listicles skip. The companion warning — "hiding a Publish button does little if the agent can use the same credentials through another tool" — shows a real operator's paranoia: check the whole route, and check both that allowed actions work and forbidden ones are blocked.

The essay is also honest about the costs of its own advice. The cheap-model recommendation isn't "go cheap"; it's "measure the cost of an acceptable result," which can justify the expensive model when repairs would otherwise compound. And the author explicitly retracted a former position — dismissing multi-agent systems as anthropomorphising — which is rarer than it should be.

The one soft spot is that the essay doubles as course marketing (the Maven course with Hugo Bowne-Anderson closes the piece), so the advice skews toward the solo practitioner and small team; organisational mechanics get a gesture rather than a treatment. Still, the five questions in the closing are a genuinely reusable checklist.

## How this fits in the wiki

- This is the same boundary argument as [[A Convention Is Not a Constraint]] — where that piece demands you *point at* the thing enforcing a promise, mistake five names the failure mode of pretending a sentence is that thing. The two strengthen each other: conventions are guidance, constraints need an enforcer.
- The mistake-five argument overlaps with [[Fences, not Sandboxes]], which argues containment must be structural because models make bad decisions regardless of instructions — this essay gives the same idea a practitioner's framing (capability vs authorisation) without Yegge's maturity-curve optimism.
- Against [[How to Build an AI Software Factory]], this essay is the *personal* factory counterpart: Toub's piece builds the organisational machine for absorbing agent output, while this one targets the habits of the single operator standing inside it.
- The model-choice advice nuances [[When Smaller Models Win]] — both reject "biggest model by default," but intellectronica grounds the choice in retry-and-repair economics ("a cheaper attempt that needs repeated repairs may cost more overall") rather than fine-tuned task fit.

---
*Sources: [[raw/five-mistakes-people-make-with-agentic]], [[summary/five-mistakes-people-make-with-agentic]]*
*Last updated: 2026-10-14*
