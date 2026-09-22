# In Praise of Normal Engineers

Charity Majors' Craft 2025 talk (Budapest; agenda title "Build Boring Engineering Orgs," pivoted to "In Praise of Normal Engineers") argues that the 10x-engineer debate is a category error: teams own software, everyone shares the same delivery pipeline, and the right target of optimization is the org's floor — the systems that let ordinary engineers move the business every day — not its ceiling of superstars. The prescriptive core is platform engineering for humans (short deploy intervals, engineers own their code in production, observability, internal tools with SLOs, mixed-level learning teams), and the philosophical core is that excellence is minted, not rented: "zero babies were born good at algorithms."

---

## What the talk argues

- **Concede the 10x claim, then deflate it.** Some engineers genuinely are dramatically more productive ("my reaction is more: so what?"). The observation is not threatening because individual prowess doesn't set the pace of delivery — the shared pipeline does. "If you must 10x something, 10x this: build 10x engineering teams."
- **Superstar worship lets leaders off the hook.** Pedigree-first hiring (ex-Google, ex-FAANG; "top 10% of global talent" — she calls out Netflix and Coinbase) is the easy path. The hard, high-skill path is building socio-technical systems "where less experienced engineers and normal people can convert most of their effort and energy and hard work into actual product and business momentum."
- **The org is defined by its floor.** The best orgs are ones "where you don't have to be a world class engineer just to move the product, the business forward day by day." If daily progress requires staff-plus engineers, "there's something deeply wrong with your organization."
- **Design for normal people.** Normal people have confirmation and recency bias, zone out, stop reading repeated text blocks, and are worse at 2am than midday. Systems navigable by normal engineers free surplus brilliance to be "poured into the product itself" instead of "wasted on navigating the systems" — her evidence: world-class engineers at companies with six-month deploy lag whose brilliance went to "fighting with their own system."
- **The feedback-loop doom spiral.** Slow deploys → longer diffs → slower reviews → paged projects → bundled, infrequent deploys that "break more often than not" → more SREs, managers, JIRA tickets → more waiting. Her probe: a cool little app whose company turns out to have 800 engineers — "doing what? Probably waiting on each other to get shit done."
- **The fix is a platform-engineering checklist.** Deploys as the company's heartbeat (Intercom's phrase, quoted approvingly); platform engineering defined as "treating engineers like people"; every engineer deploys and owns their code (feature flags; the dev/ops split "cut in half" the feedback loop); observability because poor sense-making is "the dark matter of software engineering"; an internal tools team with internal SLOs ("anytime it takes over an hour for us to run tests and deploy an artifact to production, we invest into paying that down").
- **Diversity is operational resilience.** "A monoculture can move faster than any other team" — until someone gets sick, pregnant, or you hire outside the culture, "and suddenly everything falls apart." Teams already used to mixed genders, ages, backgrounds, geographies "can roll with it." Replace "culture fit" with "what angles or axes of diversity do they bring to this team?"
- **Mint, don't rent.** Honeycomb engineers who couldn't command FAANG salaries on arrival could "waltz right out the door tomorrow and make at least half a million dollars a year" a few years later. Excellence is learnable, situational ("excellence for a security consultancy is very different from excellence for a 12-person VC-backed startup doing payments"), and hiring only pre-minted talent replicates society's inequities.
- **The Fundamental Attribution Error, applied to companies.** We over-credit individual agency and miss the systems that shape behavior. Flip it: "Your company is a system. What kind of engineers are minted by your system?" Investments in up-leveling are reusable — they pay off at every join, team change, and new skill.
- **Hire the right people, not the best people.** Excited about your problems, good teammates, good communicators. Honeycomb's take-home isn't the interview; "the interview is the conversation" about your decisions with future peers. Communication predicts collaboration — a choice that paid off when everyone went virtual.
- **Q&A residue:** the conservatism gradient ("the closer you get to laying bits down on disk, the more conservative you should be" — databases and OSes are "ooh," dev tools and frontend are "where you get to fuck around and find out") and the Golden Path model (official SRE-supported stack; veer off it and you support it yourself) are the only boring-tech content, and both arrive in a one-minute answer.

## Key quotes

> "Any jackass can build an org where the best engineers in the world get shit done."

The core indictment, and the talk's emotional center. Superstar hiring is the easy path dressed as the ambitious one; the actually impressive skill is system-building, and it "takes more skill, more caring" out of leaders.

> "Zero babies were born good at algorithms. Great engineers are forged, they're made, they're not born."

Her rebuttal to the industry's smart-kid identity politics. She extends it: even certifiable geniuses on one axis are normal on others, and "normal encompasses a wide range of neurodiversity."

> "If it takes the slowest engineer at your company five hours to ship one line of code, it's going to take the fastest engineer at your company 5 hours to ship a single line of code."

The argument for the team as the unit of ownership and leverage in one sentence. Teams are also risk management: individuals get sick, take vacations, leave — that should not risk the business, which is why individual ownership is correct only at the six-month-survival-horizon startup stage.

> "People who self-identify as 10x engineers are almost all raging assholes."

Why the meme provokes rage, and why the best engineers she knows are humble. She diagnoses the 10x obsession as insecurity — and links it to hiring-brag rhetoric: "are you ready, willing and able to pay them salaries in the top 10% or top 1%?"

> "Talent may be evenly distributed across populations. Opportunity is not."

The equity argument smuggled inside an efficiency argument: over-indexing on already-minted talent isn't just bad strategy, it "reinforces and replicates all of the prejudices and inequities of the world at large."

> "Platform engineering is about treating engineers like people."

Her one-line definition of the movement: classic product development and design thinking applied even though "your customers are internal engineers." The fastest way to ship a line of code should also be the easiest.

> "The closer you get to laying bits down on disk, the more conservative you should be."

The Q&A's answer to boring-tech-vs-growth — a gradient, not a rule. Notably this is the only technology-selection guidance in a talk billed as "Build Boring Engineering Orgs."

> "Your company is a system. What kind of engineers are minted by your system?"

The Fundamental Attribution Error turned into an org-design instrument. It reframes hiring as a lagging indicator of the system rather than a strategy in itself.

## Key themes

- **#concept — Floor over ceiling.** Org quality measured by what ordinary engineers can ship, not by the peak talent it can attract. The inverse test: needing staff-plus engineers for daily progress is an org defect.
- **#pattern — Feedback loops as org structure.** Deploy interval, review latency, ownership of production code, observability, internal SLOs — all one variable (loop length) with a compounding sign: systems feed on themselves "going downhill or... going uphill."
- **#concept — Minted vs. rented excellence.** Excellence as learnable, situational, and specific; the org as the mint. The bootstrapping question (who mints the minters?) is left open.
- **#person — Charity Majors.** Co-founder and CTO of Honeycomb, pioneer of modern observability, co-author of *Observability Engineering* and *Database Reliability Engineering*. This talk shows her org-design hat; her observability-work hat appears in [[The Three Pillars of Observability]].

## Opinionated take

The talk's best move is rhetorical honesty: by granting that 10x engineers exist, Majors strands the superstar-hiring position with no defense except habit. "So what" is a devastating answer because it's empirically grounded — the shared pipeline argument is the strongest single sentence in the talk, and it converts an argument about talent into an argument about systems, where leaders actually have leverage. The Fundamental Attribution Error framing is the right sociological tool, and "what kind of engineers does your system mint?" deserves to be asked in every hiring postmortem.

The weak joints are the ones the gist's own digest flags honestly. The doom spiral is diagnosed vividly and treated only with the same prescription that causes the problem ("keep the deploy interval short") — there's no migration path for the 800-engineer company already trapped in it. The compensation contradiction is real: she skewers top-10% hiring language, then admits Honeycomb mints engineers worth $500k+ at FAANG and retains them on fulfillment alone, without saying what happens when that stops being enough. "Business impact" is declared "the only really meaningful measure of productivity" and never operationalized — the same vagueness she condemns in "culture fit." And the "who thrives here?" analysis carries a bias trap she names and then waves off as "complicated."

The loudest omission is AI. A 2025 talk on productivity, hiring, and "doing the reps" that never mentions AI is either presciently orthogonal or conspicuously dated, depending on how fast the assumptions it rests on dissolve: individual productivity multipliers are precisely what AI claims to be, junior engineers learn by doing the reps AI now does for them, and "normal engineer" is being renegotiated in real time. Her floor-over-ceiling thesis may actually strengthen in an AI-saturated org — the bottleneck migrates to whatever the pipeline makes hard for normal people — but the talk can't say so, because it never looks.

## Related pages

- [[Shaped by Demand — The Power of Fluid Teams]] — Dan North's sibling Craft 2025 talk makes the same team-level turn from the structural side: both argue the team, not the individual, is the unit that delivers, and both treat org structure as a system to be engineered rather than a ladder of talent. Majors supplies the hiring-and-culture rationale North's demand-led mechanism presumes.
- [[Building World-Class Engineering Teams in the Age of AI]] — Rajan and Dohmke describe AI-era teams through metrics (89% more PRs) and role collapse; Majors complicates that picture by locating excellence in the system that mints people, not the people, and by never mentioning AI at all — a 2025 talk whose silence is now the most interesting thing about it.
- [[Nicole Forsgren on AI and Developer Productivity]] — Forsgren's inner-loop/outer-loop and cognitive-load diagnosis is the same argument Majors makes pre-AI: the pipeline, not the producer, sets the pace, and "cognitive carrying cost" of slow deploys is cognitive load wearing org clothes. Forsgren supplies the measurement framework Majors gestures at with "business impact."
- [[The Three Pillars of Observability]] — Majors' other hat. Her "poor sense-making is the dark matter of software engineering" is the org-level case for the observability her company sells; this talk is what the vendor-CTO says when she's not selling storage.

---
*Sources: [[raw/in-praise-of-normal-engineers]], [[summary/in-praise-of-normal-engineers]]*
*Last updated: 2026-09-13*
