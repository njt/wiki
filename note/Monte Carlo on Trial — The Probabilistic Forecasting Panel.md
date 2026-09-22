# Monte Carlo on Trial — The Probabilistic Forecasting Panel

A verbatim transcript (with its own pre-written digest) of a conference panel — almost certainly Craft 2025, given the speaker cohort — where Daniel Vacanti and Colleen Johnson defend Monte Carlo forecasting over historical throughput data against a trained statistician (Gaia Becheri), a lean skeptic (Nigel Turlow), and a moderator (Dan North) who keeps forgetting to be neutral. The panel never resolves anything, and that is precisely its value: it is a rare public record of a sold method failing basic cross-examination on its own definitions, ending with Vacanti conceding "the panel has failed" because nobody agreed what "distribution," "stability," or "predictability" mean.

---

## Key Quotes

> "It is impossible to manage software development without managing risk and it's impossible to manage risk without some type of forecasting."

Vacanti's opening position. North immediately flagged the second half as "a bit racy" — rightly, because it smuggles the whole debate in: if forecasting is *necessary* for risk management, then interrogating the forecast interrogates risk management itself. Nobody picked up the smuggling.

> "There is always a distribution… You are assuming that the distribution is actually completely represented by the past data which is called the empirical distribution."

Gaia dismantles "there is no distribution" in one sentence of elementary statistics. Resampling your own throughput is non-parametric statistics with an empirical-distribution assumption, and such estimators "only converge when you have enough data" — which is why the field's "few weeks of data" claim is suspect. Vacanti's on-record reply to this, the strongest technical challenge of the session: "I really did not understand any of that."

> "Those are made up. Those are mathematical hacks." — Vacanti, on Gaussian/normal/log-normal distributions.

Gaia's rebuttal is the adult in the room: "a model is a made up construct to explain reality… The point is whether it's useful or not." Vacanti's bluster and her calm correction are the panel in miniature.

> "Asking about a single data point, it's like asking about a single surgery outcome. Is the patient going to die? We don't know yet. We haven't done the operation." — Dan North

North's core misapplication charge: Monte Carlo answers population questions ("what's the likelihood that 10 of a hundred projects make it?") and gets deployed as a single-case oracle ("will this project finish by June?"). The practitioners never rebutted it — which is a shame, because a rebuttal existed (see analysis below).

> "I have never met an adult professional outside of the stochastic academic world that could tell you what the difference is between an 80% and an 85% confidence interval." — Dan North

And Gaia, who lives with risk-trained executives, confirms from the field: "They will just tell me why you didn't succeed. You told me it would be 85% of probability of success." Insurance translates the 99.5th percentile into "1 in the 200 years event because this they understand." Percentiles don't survive the trip to the executive floor.

> "I think the focus shouldn't be on getting a probability of when this work will be done. I think the focus should be on why are we having to do this to get a probability of when the work should be done." — Nigel

The lean counter-position in one line: the variation that makes forecasting necessary comes from "management, red tape, bureaucracy" — fix that and you may not need the simulation at all. Vacanti's answer was to go "all Princess Bride" — "I don't think you keep saying that word. It doesn't mean what you think it means" — and to settle the definition question by authority: "If I have to choose between listening to Nigel or listening to Dr. Shewhart, I can tell you who I'm going to listen to."

> "Fundamentally where this panel has failed is before we can even have an argument, we have to agree what we're arguing about."

Vacanti's own verdict, in the session's most honest moment. Distribution, stability, predictability, stationarity: all contested, none resolved. North's salvage attempt — a system can be "super spiky in a predictable way," so predictable without stable ("the contrapositive is not true") — was dismissed rather than engaged.

> "I don't have to justify my professionalism in statistics. I mean, you can just look at my CV." — Gaia's closing, after being told to "go practice" with Monte Carlo by the people whose position she had just statistically dismantled.

Colleen's most pragmatic line deserves the counterweight: "If really what we're trying to do is help our developers spend more time coding. That's really the goal, not to have perfect dates." And the moderator opened with "Just for the record, I haven't got a clue what I'm doing here. So there we go. I'm here to learn" — which aged ironically given how much of the arguing he ended up doing.

## Key Themes

- **#concept — The empirical distribution is an assumption.** "It's just your data" is not assumption-free; it assumes your past data fully represents the future distribution, and non-parametric convergence needs data volume the field hand-waves about ("few weeks"). Monte Carlo proper, Gaia notes, "requires to have a distribution or a stochastic process… as an assumption" — the resampling-vs-modeling distinction was never made explicit on stage.
- **#concept — Stable is not predictable.** North's distinction: stable means within Shewhart's bounds of variation; predictable means you know the system's characteristics and what makes it change, even if it's spiky (his example: fractals). "It can be predictable without being stable." Gaia adds *conditional predictability*: a non-stationary system with known dynamics (holidays, seasons) is still predictable. Vacanti conceded only Shewhart's and Little's definitions, and conceded the panel had failed to agree.
- **#pattern — The forecast that becomes a target.** Nigel: clients generate an 85th percentile, it hardens into a commitment, then a rerun says 40% "but the management don't want to hear that now… that's more of a management problem than a tooling problem or a math problem." Colleen's reframing — a falling percentile is an invitation to ask "what changed?" (throughput, capacity, scope) — is the best available answer, but the organizational refusal to hear updated probabilities was left hanging.
- **#tool — The practitioner's toolkit.** Monte Carlo over throughput, no story points; daily reforecasting ("every day that goes by is a new data point"); like-for-like sampling ("don't use data from December… to forecast a project that you're delivering in March"); WIP over headcount ("the amount of work in progress… is going to probably impact your throughput more than shuffling people around"); the "1 in 200 years" translation of extreme percentiles; spreadsheet-level entry ("You can randomize your data and start to see how much variability there is").
- **#person — The cast.** Daniel Vacanti (flow-metrics author, sells the method), Colleen Johnson (ProKanban.org CEO, operational optimist), Gaia Becheri (statistician in insurance risk, the session's technical conscience), Nigel Turlow (lean consultant, disclosed shared clients with Vacanti and prepped by feeding Vacanti's books to Grok and ChatGPT), Dan North (moderator turned combatant).

## An Opinionated Read

This transcript is valuable precisely because it went badly. Vacanti's position contains a defensible core — resampling your own throughput is not the same as assuming a parametric family, and "the future can be modeled with data from the past" is the same assumption every engineering discipline makes — but he defended it by denying the word "distribution," getting corrected by an actual statistician, then pleading audio problems. The method's public advocate failed a basic exam in public, and the record shows it.

North's single-case critique was the strongest unanswered charge, yet a rebuttal existed that nobody made: in flow forecasting the unit of analysis is the work *item*, not the project — a project is a population of items, so "when does this finish?" is a portfolio question after all. Had the practitioners articulated that, half the fight dissolves. Instead Vacanti reached for authority (Shewhart, Little) over argument, which is how you lose a room that came to argue.

Nigel's dilemma is sharp but rests on a false dichotomy: stability is not binary, and the interesting regime is the mushy middle — non-stationary but slowly drifting systems — where plain averages lie and resampling genuinely helps. Nobody made that argument either. What the panel actually demonstrated is Gaia's unanswered question: "what are you trying to estimate?" The purpose question precedes the method question, and an entire debate about assumptions happened with zero debate about evidence — no backtesting, no calibration, no "why 85%?" Meanwhile the commercial elephant (the defenders sell books, training, and certification) sat unacknowledged in the room.

For an agent-era wiki this is uncomfortably current: agent fleets are WIP-explosion machines and probabilistic claims are spreading everywhere, but this panel is recorded evidence that even risk-trained professionals cannot operationalize a confidence interval in a meeting. The tooling is the easy part; the vocabulary was never agreed and the validation never done.

## Related Pages

- [[Mind Your P's and Queues]] — Vacanti's own Craft 2025 talk supplies the method this panel stress-tests: its Monte Carlo demonstration and Shewhart/Little reading path now sit next to the transcript of the same author being cross-examined on exactly those foundations, which complicates the talk's clean "prioritization is waste" arc with the contested ground beneath it.
- [[Shaped by Demand — The Power of Fluid Teams]] — North's Craft 2025 talk shows him as the demand-side systems thinker; here he is the moderator who abandons neutrality to prosecute his "Monty Python Simulation" critique, and his predictable-without-stable distinction is the one genuinely new analytical tool this session produced.
- [[Reducing Risk in Projects, Increasing Resilience]] — Nigel gestures at "Dave… he's a complexity guy" in the audience; Snowden's manage-the-preconditions-not-the-outcomes thesis is the position Nigel is reaching for, and this panel shows what happens when the forecasting mindset meets it without shared vocabulary.
- [[Probabilistic Engineering and the 24-7 Employee]] — Davis declares the industry's contract now probabilistic ("correctness is a belief with a widening confidence interval"); this panel complicates that by showing the communication layer such a world presumes — executives who understand "15% chance it's later" — does not exist even among risk professionals.

---
*Sources: [[raw/monte-carlo-on-trial-the-probabilistic-forecasting-panel]], [[summary/monte-carlo-on-trial-the-probabilistic-forecasting-panel]]*
*Last updated: 2026-09-13*
