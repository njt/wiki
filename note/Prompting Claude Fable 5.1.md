# Prompting Claude Fable 5.1

Anthropic's own field guide to what changed between Claude Fable 5 and 5.1, structured as a symptom-to-fix map: each section starts from an observation ("prose runs long and dense," "turn ends before the work is done") and ships a ready-to-paste prompt fragment. It's the missing companion piece to the "simplify your context" doctrine — where [[New Rules of Context Engineering]] argues you can delete 80% of the system prompt, this doc catalogues the handful of instructions that *still* earn their tokens.

---

## Key Quotes

> "Effort is the primary control for trading off intelligence, latency, and cost on Claude Fable 5.1. Re-run the sweep even if you already ran one on Claude Fable 5: effort level names don't correspond to the same amount of thinking across models."

The single most actionable line in the doc, and a quiet warning against treating effort labels as stable. The same name now buys a different amount of thinking — which means every hard-won effort benchmark from the Fable 5 era is stale. This is the model-vendor version of what [[Controlling Reasoning Effort in LLMs]] maps from the research side: the knob is real, but its calibration is not portable.

> "Mannered prose substitutes metaphor and flourish for direct statement. Instead of 'a parameter worth varying,' the mannered writer produces 'a dial worth turning.' … The fix is to say what you mean. When a literal phrase is available, use it."

Anthropic shipping a *definition of bad prose* into its system-prompt guidance is notable. This is the vendor's own name for the tic catalogue that [[Why Does AI Write Like That]] and [[LLM Cliché Highlighter]] diagnose from the outside — and the fix is the same one Kriss lands on: precision over performance. But where Kriss treats it as an aesthetic tragedy, Anthropic treats it as an engineering input — an anti-pattern you can describe and then suppress.

> "Append each assistant turn to the history exactly as the API returned it, thinking blocks included, and don't edit earlier turns between requests."

The doc's most load-bearing mechanical rule. Thinking blocks are now cryptographically bound to the exact conversation that produced them; editing a prefix mid-session returns a 400. This converts a long-standing best practice (append-only history, for cache and coherence) into a hard constraint for new accounts — and it's the mechanism-level justification for the "then → now" reversals in [[New Rules of Context Engineering]]: you *must* send per-turn reminders as turn-scoped system messages rather than rewriting the system prompt.

> "Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done… do that work now with tool calls."

The autonomy nudge. Fable 5.1 can do very long tasks, but its default sometimes *narrates* the next step instead of taking it ("Next, I'll…", "Shall I apply this?"). The fix is a system-prompt block whose first sentence — "the user is not watching in real time" — carries most of the effect. It's a concrete, pastable answer to the chronic under-delivery that [[Agent Coding Workflow]] and [[Human-in-the-Loop is Tired]] both flag from the practitioner side.

> "Keep changes to what the request needs. Something else you notice worth doing… is a suggestion to make at the end, not a change to make."

The mirror-image failure: not under-delivery but over-delivery. Fable 5.1 will fix nearby bugs, extend unrequested behavior, and commit more tests than the change warrants. Anthropic's fix is to *define the request as the scope of the deliverable* — the same "scope is the deliverable" discipline the wiki's specification-first thread pushes in [[Specifications as the Product]].

## Key Themes

- **#concept effort levels are not portable** — the same label buys different thinking across model generations. Re-benchmark; don't inherit. Step *down* to `medium`/`low` where evals hold, not just up.
- **#concept thinking blocks are conversation-bound** — append-only history moves from best practice to enforced contract; prefix edits invalidate the chain. Turn-scoped system messages (`clear_at: "next_user_message"`) replace mid-session rewrites.
- **#pattern symptom-to-nudge prompting** — every section is a failure mode with a verbatim, paste-ready instruction. This is the vendor formalizing "prompt engineering" as *reactive patching of observed behavior*, not proactive system design.
- **#pattern the anti-pattern definition** — to suppress a behavior, name its anti-pattern precisely ("mannered prose"), then forbid it. Mirrors [[Guardrails and Feedback Loops]]' "linters beat prompts," except here the prompt *is* the linter, because prose style can't be linted deterministically.
- **#tool turn-scoped system messages** — beta (`mid-conversation-system-clear-at-2026-08-21`) mechanism for per-turn instruction that auto-clears, keeping history append-only while still nudging every turn.
- **#comparison under-delivery vs. over-delivery** — the doc ships paired fixes for opposite scope failures: one block to stop the model asking permission, another to stop it exceeding the ask. Autonomy cuts both ways and needs both brakes.

## Critical Analysis

**This is documentation doing prompt-engineering's job — and mostly succeeding.** The format (observe a symptom → paste a fix) is genuinely the right granularity for a model-release note. But it's also a confession: Anthropic is patching its own model's behavior with *prompts*, shipped to customers as if they were hotfixes. That's the same "instruction in context is not a constraint" problem [[claude-ctrl]] names — the vendor is handing out context strings as substitutes for training-time or product-level fixes. Some of these (writing density, over-delivery) are arguably things the model should just *not do*, not things every customer should have to tell it not to do.

**The append-only requirement is the only structural change, and it's under-explained.** The doc tells you the *what* (thinking blocks are bound to the conversation) but not the *why* or the *cost*. It's clearly a cache/coherence mechanism, but the interaction with client-side compaction is left as "experiment with later compaction points." For anyone running a custom harness — precisely the audience that hits the batching and compaction sections — this is the part that will actually break production, and it gets the fewest words.

**The two scope failures are the same failure in opposite directions, and the doc never says so.** Under-delivery ("asks permission for work already requested") and over-delivery ("fixes nearby code the task didn't mention") are both failures to treat the request as a boundary. Shipping two separate prompt blocks obscures that they're one discipline — scope as deliverable — which is the central insight of [[Specifications as the Product]]. The wiki's synthesis is cleaner than the vendor's here.

**The writing-density section is the most revealing.** That Anthropic's *official* answer to dense prose is "tell the model what mannered prose is" confirms the diagnostic apparatus [[Why Does AI Write Like That]] and [[LLM Cliché Highlighter]] build from the outside. It also inverts the usual polarity: here the vendor is *acknowledging* the prose problem rather than denying it, and the fix is a definition of the anti-pattern — a prompt that reads like a style guide. The short version ("Please remove all mannered prose") is the real tell: the model already knows what mannered prose *is*; it just doesn't know you don't want it.

**What's missing:** nothing on failure to *recover* — the retry/error loops get one line buried in the autonomy block — and nothing on evaluation. The doc is pure prompting guidance with no evals cited for the claimed "drops substantially" on unrequested test code. Treat the specific fragments as strong starting points, not measured results.

## Connections

This is the companion to [[New Rules of Context Engineering]], which ends by pointing to "the companion Fable field guide" without linking it — this is that guide. It shares the writing-prose diagnosis with [[Why Does AI Write Like That]] and [[LLM Cliché Highlighter]], the effort-knob frame with [[Controlling Reasoning Effort in LLMs]], the under-delivery problem with [[Agent Coding Workflow]] and [[Human-in-the-Loop is Tired]], and the scope-as-deliverable discipline with [[Specifications as the Product]]. For the mechanism of instruction delivery (where turn-scoped system messages sit among the other levers), see [[Steering Claude Code]].

---

*Sources: [[raw/prompting-claude-fable-5-1]], [[summary/prompting-claude-fable-5-1]]*
*Last updated: 2026-09-04*
