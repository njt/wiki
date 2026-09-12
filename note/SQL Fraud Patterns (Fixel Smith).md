# SQL Fraud Patterns (Fixel Smith)

A program-integrity analyst's field guide to catching transaction fraud with SQL, not ML. Six composable patterns — velocity, impossible travel, amount anomalies, suspicious merchants, off-hours, and window-function primitives — that together form a scoring system where failing 3–4 signals means fraud and failing one means your grandma being weird with her debit card on vacation.

---

## Key Quotes

> "Fraud detection in transaction data is mostly SQL, not machine learning, not graph databases, not whatever Gartner is hyping."

The article's thesis statement. Smith isn't saying ML is useless — he's arguing that the operational reality of fraud detection is analysts writing queries against transaction tables. The fanciest model in the world is worthless if your analysts can't iterate on hypotheses in hours instead of weeks. This is the #tool-not-religion take that the HN thread mostly missed.

> "A transaction that fails on three or four of them is almost always fraud. A transaction that fails on one might be your grandma being weird with her debit card on vacation."

The compositional insight: individual signals are weak, but their intersection is strong. This is the same logic behind ensemble methods in ML, but expressed as SQL `WHERE` clauses. No model training, no feature engineering pipeline — just queries you can read.

> "Real cardholders almost never buy something for exactly $1.00. Their coffee costs $4.73. Their gas costs $52.81."

The roundness-as-signal observation is the kind of thing you only notice by staring at real data. It's domain intuition crystallized into a filter. Also the most controversial pattern in the HN thread — non-US commenters pointed out that VAT-inclusive pricing makes round numbers normal in Europe, which reveals the US-centric assumption baked into the heuristic.

> "Three filters. That's it."

On Pattern 6 (window-function primitives) reducing a card-testing-ring detection to `tx_of_day >= 5 AND time_since_last < INTERVAL '60 seconds' AND merchant_change = 'changed'`. The real product isn't the fraud detection — it's the iteration speed. When analysts can express fraud hypotheses as SQL filters instead of engineering tickets, the loop shrinks from weeks to hours.

> "Auto-blocking on a single rule is how you lose customers."

Buried in the "Things Left Out" section but should be the headline. Smith understands that these are signals, not verdicts. The system design question isn't "which pattern catches fraud" — it's "what happens when a pattern fires on a legitimate transaction?"

---

## Key Themes

**#pattern Compositional fraud detection.** Each pattern alone is weak. Velocity catches vending machine operators. Impossible travel misses same-metro fraud. Amount anomalies don't apply to benefits programs. But stack them and the false-positive rate collapses because different patterns fail for different reasons.

**#pattern SQL as analyst interface, not just query language.** Pattern 6 is the real insight: materialize window-function columns (`time_since_last`, `merchant_change`, `tx_of_day`) and fraud rules become `WHERE` clauses. This isn't about SQL's technical merits — it's about who can write fraud rules. Analysts can write SQL. They can't write Python services.

**#concept Baseline, don't threshold.** Pattern 4's static-threshold version is bad ("A Costco does that in 90 seconds. A used bookshop, never."). The rolling-baseline version is good — compare each merchant against its own history, same hour, same day-of-week. This is the same principle as [[Anomaly Detection]]: what matters is deviation from expected, not deviation from a magic number.

**#tool Window functions as composable primitives.** `LAG`, `ROW_NUMBER`, `RANGE BETWEEN`, named `WINDOW` clauses. Smith treats these as building blocks, not just query features. Once `time_since_last` exists as a column, every analyst on the team can compose it into new fraud rules without touching the pipeline.

**#concept Signals over models.** The whole article is an argument for human-readable signals over black-box models. Not because signals are more accurate — because they're debuggable. When a transaction gets flagged, you can trace exactly which patterns fired and why. With an ML model, you get a probability score and a shrug.

---

## Critical Analysis

**What's right.** The compositional approach is genuinely good. Six simple patterns, each expressed as SQL, each with known failure modes, combined through scoring rather than AND logic. This is exactly how production fraud systems work — they just have more patterns, more data, and real-time infrastructure. Smith is describing a reasonable starting point, not a toy.

**The HN thread revealed the real problem, and Smith mostly sidesteps it.** The comments tore into the round-number heuristic (VAT-inclusive pricing in Europe, "round up to donate" producing $51.00 totals, coffee priced at exactly $3.50). Smith acknowledges the pattern is US-centric for benefits programs, but the deeper point is that every heuristic has edge cases the analyst can't anticipate. This is where the compositional model actually helps — if round-amount fires but nothing else does, you don't block the transaction. But it also means you need analysts who understand the domain well enough to tune the patterns, which may be the real constraint.

**The disclaimer matters.** Smith says upfront that "nothing here comes from anything I've actually worked on or seen." This turns the article from "here's what I do at my job" into "here's what I'd try if I were starting from scratch." The patterns are plausible and well-reasoned, but they're untested. The HN crowd picked up on this — several commenters questioned whether "Fixel Smith" is even a real person given the AI-generated adjacent content (a novel, a music album, all published within days).

**What's missing: feedback loops.** Smith mentions false positives need human review but doesn't describe the feedback mechanism. In production, every flagged transaction that turns out legitimate is a training data point for tuning thresholds. Without this, you're just guessing at threshold values. This is the same gap between [[Anomaly Detection]]'s elegant math and the operational reality of "what do you do when the alert fires." The system that improves from its mistakes is worth ten times the system that just flags things.

**The real insight is organizational, not technical.** Pattern 6 isn't a fraud pattern — it's an argument about team structure. By materializing window-function columns, Smith is saying "make the data comprehensible to analysts, not just engineers." This is the same principle behind [[Materialized Views Are Obviously Useful]]: derived data should live in the database, not in application code. It's also the [[Long Live Systems of Record]] argument inverted: the database isn't just where truth lives, it's where analysis lives.

**Compared to the broader wiki.** Smith's patterns sit at the intersection of [[Databases and Data]] and [[Software Engineering Craft]]. They're not about agentic development or AI — they're about the fundamental craft of making data answer questions. But the compositional philosophy (simple primitives that compose into complex behavior) is the same pattern that shows up in [[Guardrails and Feedback Loops]] (linters as composable enforcement), [[Harness Engineering]] (feedforward + feedback loops), and [[Compound Engineering]] (systems over manual review). Smith is doing compound engineering with SQL.

---

*Sources: [[summary/sql-fraud-patterns]]*
*Last updated: 2026-05-22*
