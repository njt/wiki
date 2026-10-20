# How to Be More Ambitious — A Tutorial for Humans with Models

Felix Rieseberg (who works on Claude) turns the vague rallying cry "we're not ambitious enough with AI" into an actionable framework: ambition comes in four types — Speed, Altitude, Depth, Range — each with concrete techniques, all grounded in the psychology of self-efficacy.

---

## The four types

- **Speed** — same work, less time. The bread-and-butter economic value; skipped as well-understood.
- **Altitude** — operate at the outcome, not the task. "Fix this crash" becomes "keep our apps crash-free." The tell: if you'd have meetings about it, altitude applies. It pays off by enabling durable, human-out-of-the-loop workflows — but only after you instrument the world so the agent can act (crash reports, commit mapping, machines on three OSes).
- **Depth** — work with the instruments of field-leading experts, and push until even they might learn something. Insist on real tools over role-play vocabulary ("when Claude reaches for markdown tables, insist on the use of databases and statistical models"), set unreasonably hard goals, adversarially check assumptions against raw data, produce new knowledge, and use an outside judge.
- **Range** — borrow fields you wouldn't have budgeted for before. Replace nouns with verbs (Kelley/Littman), then "assemble the crew."

## Key quotes

> When prompted to act like a professional, models often switch vocabulary without actually switching methods.

A sharp diagnosis of why "act like a senior engineer" prompts disappoint: the model borrows the dialect, not the toolkit. The fix — insist on the actual instruments — doubles as good general prompting advice.

> Do *not* ask the model to judge itself.

Stated flatly, in the middle of the Depth section. It connects directly to the wiki's long thread on external verification and LLM-as-judge failure modes.

> It is easy to accidentally engage in role play, which will make your transcript weirder without improving the actual outcome. ... Don't ask the model to *write* like a short seller, ask it to *do* what a short seller *does* to research their position.

The most practically useful line in the Range section, and the one most often violated by people "assembling crews" of personas.

> We humans have always been held back by what we believe possible.

The essay's pivot from technique to psychology: self-efficacy, goal-setting research (Locke & Latham), the Dantzig story, Bannister's four-minute mile. Rieseberg's claim is that the constraint on AI use is now human imagination, not model capability — a claim that fits Tao's "astounding in some directions, inadequate in others" framing he opens with.

## Themes

#concept #pattern

## Analysis

This is one of the better attempts to operationalize "be more ambitious," precisely because it refuses to be inspirational. The four-type taxonomy is doing real work: most "you're not using AI hard enough" essays silently conflate Speed with everything else, and once separated it's obvious that most practitioners live entirely in Speed. The Altitude advice is also honest about the cost — "keep our apps crash-free" only became possible after Rieseberg built the plumbing (data access, verification environments), which quietly reinforces the wiki's recurring finding that agent autonomy is purchased with harness engineering, not prompting.

The essay's blind spot is that it's written from inside the frontier-lab bubble. Its examples (planet discovery, QBR decks, Blender renders) assume a user with good data, good tooling, and taste; the "unreasonable goals" trick ("ask for 200ms") is genuinely good for breaking model timidity but has no answer for when the unreasonable goal should be abandoned rather than pursued. The Depth section's "tell the model it's very much solvable" cuts both ways — it's the same mechanism that produces confident failure in the wrong contexts. And the self-efficacy frame, while charming, is asserted more than argued: Bannister and Dantzig are stories about *humans* breaking limits, offered as analogy for a tool that doesn't experience belief at all. The interesting unexamined question is whether the model's own "self-efficacy" (its learned timidity) is the real lever, with the human's as a side effect.

Still, as a practitioner tutorial it's strong: specific prompts, an honest acknowledgment of limits ("Claude, go make money" doesn't work), and a memorable anti-pattern (role-play vs. method).

## Relations

This source strengthens [[Agency]] with a concrete taxonomy: where that page argues ambition is a choice, Rieseberg gives the four axes the choice can be made on. It nuances [[What You Bring to AI Determines the Result]] — the user's imagination is the input that matters, but Rieseberg adds that the environment's instrumentation matters just as much, since altitude is impossible without data access. And it complicates [[Finding Problems Worth Working On]] by locating the bottleneck in self-efficacy rather than problem selection: the hard part isn't finding a good problem, it's believing the ambitious version of the problem is askable at all.

---
*Sources: [[raw/how-to-be-more-ambitious-a-tutorial-for-humans-with-models]], [[summary/how-to-be-more-ambitious-a-tutorial-for-humans-with-models]]*
*Last updated: 2026-10-20*
