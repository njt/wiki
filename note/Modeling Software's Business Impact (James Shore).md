# Modeling Software's Business Impact (James Shore)

James Shore's fourth essay on assessing AI's impact argues that the hard part of AI-era ROI isn't how fast you build but *what you deliver*: since there's no guaranteed link between software cost and value, teams should forecast business outcomes with NPV models and proxy metrics (Pirate Metrics), run "product bets" through a milestone-and-ceiling funding process, and treat prioritization as the realization of company strategy — because with code cheap, the new bottleneck is the customer's ability to absorb features.

---

## What it argues

Shore opens by disclaiming the essay's own series: this part "isn't really about AI at all. It's about *building the right thing*." The insight is that value modeling is rare not because it's hard but because it's organizationally complex, while cost estimation — the thing everyone does — measures the wrong axis entirely.

The AI-specific twist is the migration of the bottleneck. Lee Eason's hypothesis: customers can only absorb so many features. Expensive development once *forced* prioritization as a side effect; now cheap generation ships work customers won't pay attention to. "In the AI era, our most impactful work could just be ignored."

## Key quotes

> "You can spend twenty million dollars developing software that's worthless, and two million dollars developing software that transforms a business. Cost improvements are typically marginal, and difficult. Value improvements are *game-changing*."

The cleanest statement of the whole essay. If AI collapses the cost axis, marginal gains there matter even less.

> "We need to treat customer attention as the scarce resource it is and only ship the things that are truly valuable."

The demand-side bottleneck is invisible in a way the engineering bottleneck never was — shipped-but-ignored work produces no feedback at all.

> "All models are wrong, but some are useful" (George Box, via Shore).

Shore's defense of the NPV spreadsheet is not accuracy but function: surfacing hidden assumptions ("stakeholders present a solution without verbalizing the problem"), enabling adversarial peer critique of forecasts, and supporting milestone-based continue/cut-losses decisions.

## Key themes

#concept #pattern — value modeling over cost estimation; proxy metrics (Pirate Metrics: Acquisition, Activation, Retention, Referral, Revenue) as the answer to the attribution problem; milestone-and-ceiling spending with Build-Measure-Learn governance.

## Analysis

The most opinionated part of Shore's series, and refreshingly non-AI: the prescription is two decades old (NPV, AARRR, Lean's build-measure-learn) and that's the point — the AI-era skill gap is *business* craft, not prompt craft. His candid war story (two years, a new CFO, "CFOs like rigor too") is the honest part most framework essays omit: the modeling is easy, the org politics is the project.

Two cautions worth keeping. First, the milestone-and-ceiling model is elegant but depends on the NPV guess it's derived from — a garbage ceiling still funds garbage. Shore admits the models rest on guesses, but the process legitimizes whatever numbers survive the adversarial critique. Second, "prioritization is strategy" is right, yet the essay's remedy is a heavyweight process for *substantial initiatives only* — the long tail of small AI-enabled features, which is exactly where the volume explosion happens, escapes the model entirely.

It pairs sharply with Cagan's take on output vs. outcomes and with the cost-side counterpoint from Beck on YAGNI: Shore attacks from the value side what Beck attacks from the options-pricing side — both say cheap code changes the economics of *deciding what to build*, not just building it.

## Related pages

- [[The AI Productivity Paradox]] — Marty Cagan's complementary diagnosis from the product side: AI accelerates output, not outcomes; Shore supplies the financial-modeling mechanism Cagan's argument implies.
- [[The Cost YAGNI Was Never About]] — Kent Beck's NPV framing of YAGNI strengthens Shore's NPV spreadsheet; Shore extends it from feature scope to portfolio prioritization.
- [[Uber — Agentic Engineering Shift]] — Uber's "unresolved measurement gap between activity metrics and revenue impact" is precisely the attribution problem Shore models his way around.

---
*Sources: [[raw/modeling-softwares-business-impact]], [[summary/modeling-softwares-business-impact]]*
*Last updated: 2026-10-03*
