# Preseen — AI Forecasting Platform

Preseen is an AI-powered forecasting startup that uses multiple independent AI agents — each analyzing questions from different analytical angles — to produce calibrated probabilistic predictions on finance, politics, and global events. In competitive forecasting tournaments (Kalshi, Metaculus, FutureEval), it has posted results that are genuinely freakish: 6th all-time on Kalshi ($35 → $1.94M), and the first bot ever to win a human tournament on Metaculus. A Good Judgment Superforecaster's verdict: "I was prepared to be underwhelmed, but the sh*t is impressive." The product is in waitlist-gated pre-launch as of mid-2026.

---

## Key Quotes

> "I was prepared to be underwhelmed, but the sh*t is impressive." — A Good Judgment Inc. Superforecaster

This is the only external validation on the page and it's doing a lot of work. Good Judgment Inc. is the gold standard in human forecasting — they're the people who proved superforecasters exist as a measurable phenomenon. Getting one of them to say "impressive" about your AI forecaster is the equivalent of Deep Blue getting Kasparov's nod. The quote is deployed with precisely the right amount of irreverence — it doesn't overclaim, it just says "we passed the sniff test from the people who matter."

> "The edge is knowing the odds before consensus forms."

The investment-professional pitch in seven words. Preseen isn't selling predictions — it's selling temporal arbitrage. If you know the real probability of a cocoa supply shock before the market prices it in, you win. The value proposition is speed, not accuracy. This is a cleaner thesis than "we predict the future" because it avoids the omniscience trap — you only need to be faster than consensus, not right about everything.

> "Independent AI scientists each analyze from different angles (docket reading, broad source sweeps, skeptical analysis). Their estimates are synthesized into a single calibrated probability."

The multi-agent architecture is the product. Not one model making one prediction — multiple agents running divergent analytical strategies, then a synthesis layer that combines them into a calibrated output. This is the same pattern that shows up in [[Agent Swarm Model Economics]] (Cursor's planner/worker tree), [[The Advisor Strategy]] (Anthropic's advisor-executor pattern), and the broader multi-agent coordination literature. The "independent" framing matters: it's a hedge against correlated errors, which is the silent killer of single-model forecasting.

---

## Key Themes

**#tool: AI forecasting.** Preseen is one of the first commercial products applying multi-agent AI architectures to the forecasting problem — not time-series extrapolation like [[TimesFM]], but structured reasoning about geopolitical and economic questions where there's no clean dataset to train on. The approach treats forecasting as an analysis-and-synthesis problem rather than a pattern-matching one.

**#pattern: Multi-agent synthesis.** The architecture — independent agents analyzing from different angles, then a synthesis layer — is the same pattern that powers the best coding agents, AI code review systems, and research assistants. It's the "wisdom of crowds" implemented in silicon. The key design choice: agents are given *different* analytical strategies (docket reading vs. broad sweeps vs. skeptical analysis), not just the same prompt with different random seeds. This is structural diversity, not stochastic diversity.

**#concept: Calibration as product.** Preseen doesn't just predict outcomes — it produces *calibrated probabilities*. A forecast that says "65% chance" should be right 65% of the time. This is harder than binary prediction and more valuable: a well-calibrated 60% probability can be priced, hedged, and bet against in ways a binary "yes" cannot. The "Scored by reality" resolution step makes calibration the metric, not accuracy.

**#comparison: Prediction markets vs. AI forecasting.** Preseen sits in an interesting position relative to the prediction-market critique mapped in [[Prediction Markets and Perverse Incentives]] and [[America Is Slow-Walking Into a Polymarket Disaster]]. Prediction markets create perverse incentives — you can profit from making bad things happen. AI forecasting creates no such incentive: the AI doesn't have money on the line. But it introduces a different risk: if the AI's forecasts become *influential enough to move markets*, you've recreated the manipulation vector through a different channel. The AI can't be bribed, but the people who act on its forecasts can be front-run.

---

## Critical Analysis

**The tournament results are genuinely impressive, but they come with a caveat the page doesn't volunteer.** Preseen is competing in Metaculus and Kalshi — platforms where the questions are selected by humans. That means the AI is being tested on *the kind of questions humans think are forecastable*. The real test is whether it works on questions humans *don't* think to ask, or questions that are too weird, too granular, or too long-horizon for human forecasters to bother with. The tournament results prove the system is competitive with the best human forecasters at human-chosen questions. They don't prove it can do things humans can't.

**The multi-agent architecture is a strong signal of serious engineering.** Amateur AI forecasting tools throw a single prompt at a frontier model and call it a day. Preseen's architecture — independent agents with distinct analytical strategies, synthesized into calibrated probabilities, with explicit awareness of resolution-source bias — suggests they've thought hard about the failure modes. The "risk that a resolver repeats the press's conflation" line is the tell: it's the kind of thing you only know to price in if you've been burned by it.

**The business model is the unanswered question.** The page targets three markets (policy, investment, insurance) but lists no pricing and gates access behind a waitlist. The Kalshi results — $35 → $1.94M — read more like a proof-of-concept than a sustainable revenue model. If Preseen is this good at forecasting, why sell the forecasts? The obvious answer (regulatory, scaling, different skills) is plausible but unstated. The alternative interpretation: the tournament results *are* the product right now — marketing for a capability that hasn't yet found its business model.

**The waitlist phase is strategically smart in a way the page doesn't explain.** Forecasting is a domain where your reputation is your product. Launching publicly before you're confident in your calibration would be fatal — one high-profile miss and you're "the AI that got [X] wrong" forever. The waitlist lets them build a track record in public (via tournament results) while controlling access to the product. It's the Tesla Roadster playbook: prove the technology in a visible, measurable way before you try to sell it at scale.

**The contrast with prediction markets is the most interesting thing about Preseen.** The prediction-market critique ([[Prediction Markets and Perverse Incentives]], [[America Is Slow-Walking Into a Polymarket Disaster]]) is that markets don't just predict — they incentivize. Preseen sidesteps this entirely by being a forecasting *tool*, not a betting *platform*. No one can profit from making Preseen's forecast wrong. But this also means Preseen lacks the mechanism that supposedly makes prediction markets work: people with skin in the game. The AI has no financial stake in being right. The tournament results suggest this doesn't matter — calibrated probabilities emerge from good analysis, not from financial incentives — but that's a finding that undercuts the entire theoretical foundation of prediction markets. If an AI with no money on the line can out-forecast traders with real stakes, what exactly is the "wisdom of crowds" adding?

---

*Sources: [[raw/preseen]]*
*Last updated: 2026-07-29*
