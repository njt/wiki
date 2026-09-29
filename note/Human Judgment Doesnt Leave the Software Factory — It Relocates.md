# Human Judgment Doesn't Leave the Software Factory — It Relocates

Addy Osmani's field report on running a lights-on software factory, arguing that automation removes humans from parts of the loop where deterministic signals are stronger, while concentrating them on intent, system shape, quality bar, and the places where automated back-pressure goes weak. The title's thesis: judgment is relocated, not eliminated — and a good factory is measured by how intelligently it *places* human attention, not by how completely it eliminates it.

---

## Key quotes

> The percentage of code physically typed by humans may fall dramatically. I don't think human ownership needs to fall with it.

The spine of the piece. Five things someone still does: choose the problem, choose the architecture, set the quality bar, decide which verification signals deserve trust, decide when evidence is sufficient to ship. This is a more operational version of the claim in [[Cognitive Debt]] — and a direct counter to the "humans leave the loop" framing.

> Just because a software factory is showing that everything is green doesn't mean that it's actually green.

His authentication-provider example is the sharpest in the piece: asked to add GitHub auth, the agent found the UI only had room for three providers, dropped one — the one customers actually wanted — and everything stayed green. Verification shape is the actual product; check counts ≠ quality, and you must deliberately tune checks for signal-to-noise.

> Code often preserves a decision that was made, but not why the decision was made.

After merging a favoriting feature whose tests passed, he returned days later and couldn't explain how it worked. Five or ten parallel sessions mean several mental models going cold at once — context switching, which we already knew was expensive, amplified into comprehension debt. His fix is to have agents store their trajectory and reasoning as an artifact, local or committed, rather than trusting session memory or recall.

> A manual run isn't finished when the factory stops but when the human knows what to do next.

On Vercel's success/flawed/blocked/manual taxonomy: he pairs it with per-stage timing because the taxonomy hides cost — in his 82-minute run, a quick finder took 7 minutes with no rejections while favorites took 56 with two rejections and a human decision in the loop. Same factory, wildly different price per outcome.

> In my experience, you can get surprisingly far with your stock coding harness!

The tempering note most factory essays lack: a factory earns its keep only when work must be repeatable and event-driven — queues, locks, handoffs, and stopping production when human review falls behind. Warp's four-label triage (ready-to-implement / ready-to-spec / needs-info / wait-to-implement) is praised because the label is simultaneously queue, lock, and a place to park work without saying no.

## Themes

- #concept — relocation of judgment: human attention as the scarce resource the factory optimizes around, not an artifact to remove
- #pattern — verification budget, analogized to performance budgets: fast checks early (lint, types), heavy checks late (mutation, browser, security)
- #pattern — run taxonomy (flawed/blocked/manual) paired with per-stage timing so cost is visible
- #tool — his Factory repo and 82-minute demo run as a reference implementation anyone can inspect

## Analysis

This is the most honest software-factory essay in the wiki's collection because it spends as much time on the cost ledger as the architecture. The 7-minute vs 56-minute contrast is the number every factory pitch omits: taxonomy without timing is marketing. And the favoriting-feature anecdote is the strongest first-hand evidence yet that green tests and comprehension have fully decoupled — he *approved* the change and still couldn't explain it, which is precisely the failure mode [[Cognitive Debt]] predicts from the outside.

Where I'd push back: his claim that humans can't realistically read all generated code is doing a lot of work, and "we're not building rockets a lot of the time" is a dangerous default — the blast radius varies enormously, and he knows it (his own production client work has "real beefy risks"). The relocation thesis is sound; the implicit triage of where judgment is needed still rests on taste, which is exactly the thing the factory can't encode. The piece is also quietly a critique of the factory hype it participates in: "you may be fine" is repeated like a mantra, and the most useful parts (labels-as-locks, verification budgets, handoff design) all apply *before* you ever build a factory.

It also complements [[Human-in-the-Loop is Tired]] from the other direction: Summers documents supervision fatigue as burnout; Osmani responds with an allocation strategy — spend the scarce attention where back-pressure fails — which is a mitigation, not a refutation.

## Relations

- Strengthens [[Cognitive Debt]] with first-hand evidence: a merged, green-test feature the author couldn't explain, plus the trajectory-artifact fix for cold mental models.
- Nuances [[What Matters After the Software Factory Works]]: both are factory-memoir field reports, but Osmani insists the governance question is *placement* of human judgment, and adds cost-per-outcome measurement the PAAA-style writeups lack.
- Complicates [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]]: the survey treats autonomy as a level; Osmani argues it's not a single setting — trust is earned per task and per project as verification proves out.
- Extends [[Inside a Software Factory]]: Iusztin defines the factory's stages and where humans belong; Osmani adds the harness-versus-factory threshold and the queue/lock/handoff mechanics that make it real.

---
*Sources: [[raw/human-judgment-doesnt-leave-the-software-factory-it-relocates]], [[summary/human-judgment-doesnt-leave-the-software-factory-it-relocates]]*
*Last updated: 2026-09-29*
