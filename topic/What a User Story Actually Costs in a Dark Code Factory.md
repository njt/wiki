# What a User Story Actually Costs in a Dark Code Factory

The first hard, instrumented measurement of what an autonomous agentic SDLC actually spends per unit of work: $10.88 per delivered user story (median $9.56), 7.7 million tokens each, from a "dark code factory" that persists its own bill. The piece's real contribution isn't the number — it's the discovery that the number is dominated by *cache* (95.4% of tokens are cache reads; cache writes are 31.2% of cost), that a metering system is software that lies until it's audited, and that a flat-fee subscription hides the true economics behind a quota fence.

---

## Key Quotes

> "If measurement isn't part of the pipeline, it doesn't exist."

The thesis in one line. The first-generation factory shipped an 861,601-line app and cannot tell you what it cost, because it kept no records and the transcript logs expired. This is the counterpoint to every "vibe coding" story that reports output without a meter — [[Cloud Software Factories]]' "factory efficiency = shipped product / token cost" equation is unanswerable by default, not by accident. #concept

> "The factory consumed 595.7 million tokens to ship 77 stories, 7.7 million tokens per delivered story: at list prices for those models, $837.53, or $10.88 per story."

The number. The numerator includes every thrown-away token, the five FAILED stories, the retries; the denominator counts only stories that shipped. Story points barely predict cost — the medium and large bands are only 8% apart at the median — and the most expensive story ($43.24) was a 3-pointer that hit a review retry and a bugfix loop. Wall-clock mean is 2.5× the median because an overnight run stalled on a rate-limit window: a *billing* artifact, not an agent one. #pattern

> "A dark code factory is mostly a reading machine that occasionally types."

The estimate-busting finding. An agent resends the same instructions and repository context every turn; the API caches that stable context. **95.4% of tokens are cache reads** — the factory rereads ~73 cached tokens for every new one. Cache writes are only 3.3% of tokens but **31.2% of the bill**, and cache traffic overall is 77% of cost, fresh input 1.6%. The shape held across three independent samples (the factory, interactive dev sessions, and the ghost app's scraps), so it's a property of agentic compute, not one pipeline. #concept

> "Cost optimization in an agentic pipeline is cache management, not prompt shortening. Context discipline, cache-tier awareness, and orchestrators that don't stuff their own windows move the bill. Trimming your prompt wording does not."

The practical consequence, and a direct empirical confirmation of [[Managing AI Coding Costs at Scale]]'s "context bloat" insight and [[Inference Cost Napkin Math]]'s "KV-cache hit rate IS your margin." The lever is where the context comes from and how it's cached — not the prose. #pattern

> "I found a bug while dissecting the raw data that showed my meter lied. The ledger missed a sixth of real consumption, recording $694.65 against the logs' $837.53."

The most quietly radical move in the piece. When a result envelope failed validation, the controller's re-ask *overwrote* the original stage row's usage, erasing the expensive failed session from the books; crashed sessions never wrote back. 57 attempts were affected. So fxmartin filed the bug against his own factory and let its fix pipeline repair it (issue #480, PR #482). **Measurement infrastructure needs auditing just like the code it measures** — a metering layer is itself software, and software lies. #tool

> "The subscription's real currency is quota rather than money. The overnight run stalled twice on the 5-hour rate-limit window... On a flat monthly fee, time is the fence."

The "who actually pays" answer is nobody, at the margin — a $200/month Max 20x plan against ~$1,088 of measured API-equivalent work, a 5× asymmetry that reflects Anthropic's margin rather than a subsidy. The flat-fee window won't stay open; quotas tighten and tiers reprice. A metered factory notices the day the trade turns; an unmetered one "will simply feel slower and poorer, without knowing why." #concept

> "Writing the factory's specifications — its epics and stories, in interactive sessions — consumed about 190 million tokens, which is roughly 25 stories' worth of consumption... When implementation is this cheap, the code is no longer the expensive artifact."

The inversion [[Specifications as the Product]] argues for, now with a price tag. And a second self-inflicted finding: model routing was switched off the entire time, so mechanical merges burned premium-model prices on Haiku-grade work (12.3% of all tokens) — the $10.88/story figure is the *unoptimized* rate. #pattern

## Key Themes

### #concept — Metering Is Part of the Pipeline, Not an Afterthought

The difference between the first factory (unknowable cost) and the second (a $10.88/story headline) is a ledger that persists every stage attempt's tokens, cost, model, and failure category. This is the empirical, solo-developer instance of [[Cloud Software Factories]]' measurement layer, and it's what makes [[Dev Machine Foundry]]'s "565 sessions, 3,706 commits" feel like the same story told without the bill.

### #pattern — The Honest Denominator

Two ways to read failure: 5.0% of tokens marked FAILED, or ~13% counting all rework, retries, bugfix loops, and crashed sessions. Public cost claims rarely say which they use. fxmartin counts the 13% as a *quality bill* — the gates are what catch problems — but insists the number be stated honestly.

### #tool — The Meter Lies; Audit It

The ledger disagreed with the session logs, and the logs won: a validation-failure re-ask overwrote the original row's usage, silently erasing a sixth of spend. The lesson generalizes beyond this factory — any cost-observability pipeline is a piece of software with its own bugs, and "measurement" is a claim that has to be verified against ground truth, not an assumption.

## Critical Analysis

**What the piece gets right — and why it matters.** This is the missing data point the whole cost-of-AI-coding conversation has been arguing around. [[Unit Economics of AI Software]] reasons about eroding margins from ICONIQ averages; [[Managing AI Coding Costs at Scale]] prescribes from enterprise surveys; this is one person who actually *built the meter*, ran 193 story-builds through it, found the meter lying, fixed it, and published the CSVs and extraction script so every number is checkable. The methodology table (assumptions A1–A10, ground-truth-is-the-envelope, never-impute) is the credibility backbone most "I built X with AI" posts omit.

**The cache-write surprise is the most actionable finding.** Everyone knows context is expensive; almost nobody prices *cache writes* separately. That writes are 3.3% of tokens but 31.2% of cost upends the standard advice. "Shorten your prompts" targets the 0.4% of tokens that are fresh input. The real lever — orchestrators that don't stuff their own windows, cache-tier awareness, context discipline — is a harness-engineering problem, aligning exactly with [[The Knowledge Chipper]] and [[Code Cleanliness and Coding Agents]].

**Where the piece is most honest is also where it's most unusual:** it admits its own headline number is wrong twice over. Once because the meter under-counted until audited, and once because model routing was off — the $10.88 is unoptimized. Most writing sandbags the number or hides it; fxmartin treats the measurement as a first draft to be corrected. That's the ethos of [[Ways of Checking]] — "checking again re-runs the instrument; checking differently tests it" — applied to money.

**The flat-fee asymmetry is the quiet structural claim.** A $200 subscription absorbing ~$1,088 of API-equivalent work is a >5× pricing gap, and fxmartin is careful to label it a *margin* observation, not a subsidy. The durable insight is that the subscription's currency is quota and the constraint is *time* (rate-limit stalls), which means a flat-fee factory optimizes for throughput-under-quota, not for dollar cost — a different objective entirely. [[Uber — Agentic Engineering Shift]]'s "6x cost explosion" and the unresolved measurement gap live on the same spectrum.

**What's undersold.** The per-story number is one repository, one operator, one (relatively small) story size, three models. Generalizing $10.88/story to the industry would repeat the exact over-extrapolation the piece warns against elsewhere. And the "ghost application" — 696 stories at ~5.4 billion tokens, cost unknowable — is the real subject of the essay, and it stays a ghost: the most important lesson (meter before you ship, not after) is only knowable in retrospect.

**The uncomfortable coda.** The 861,601-line app was built *without* a meter and its bill is gone forever. Every reader's takeaway should be that the moment to instrument is before the first story runs — otherwise you will, literally, never know what it cost. The factory that fixes this is the one that writes its bill down as it works.

## Related

- [[Unit Economics of AI Software]] — The abstract diagnosis; this page supplies the measured per-unit number and the flat-fee-vs-API asymmetry
- [[Managing AI Coding Costs at Scale]] — The enterprise playbook; this page empirically validates its caching lever and adds the "meter lies, audit it" caveat
- [[Specifications as the Product]] — The inversion this page prices: specs consumed ~25 stories' worth of implementation budget
- [[Cloud Software Factories]] — Lloyd's measurement layer, actually built and audited by one operator
- [[The Dark Factory is a DOT File]] — The same "dark factory" family, but this page meters the factory rather than treating its config as the artifact
- [[Dev Machine Foundry]] — Schillace's autonomous clone, reported without a bill; the unmetered sibling
- [[Inference Cost Napkin Math]] — "KV-cache hit rate IS your margin" — the technical math behind the cache-read finding
- [[Model Routing Is Simple Until It Isn't]] — The routing that was (incorrectly) switched off here, and why it breaks in production
- [[Reducing Token Spend with Deterministic Workflows]] — Adam Jacob's token-reduction lever; complementary to cache management
- [[The Economic Benefit of Refactoring]] — Token reduction from the codebase side; this page argues the harness/cache side dominates

---

*Sources: [[raw/what-a-user-story-actually-costs-in-a-dark-code-factory]], [[summary/what-a-user-story-actually-costs-in-a-dark-code-factory]]*
*Last updated: 2026-09-04*
