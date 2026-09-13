# Seven Habits of a Mostly Successful Team — Sander Hoogendoorn (Craft 2025)

Sander Hoogendoorn's Craft 2025 talk on how his 13-person e-commerce team fights "technical death" — the state where maintenance eats all innovation time — through pragmatic governance, radically small steps, rule removal, and 40–50 production deploys a day. Delivered three days into repairing a serious outage from the company's biggest quarterly sale, with the seven-habit structure itself ChatGPT-generated and the talk's "automate everything" section lost to a live PowerPoint crash.

---

## What the talk argues

The diagnosis comes first: his 20-year-old deal-site employer runs an ERP eight versions behind, stores user accounts in the CMS ("CMS systems do not contain user accounts"), and syncs data with "any mechanism you can think of." But "it's never the tech... it's always the organization" — managers, product managers, and marketing demanding features while treating refactoring as a negotiable budget line. Monoliths are "okay-ish"; legacy can be beautiful ("the Pantheon has stood for 2,000 years. My software doesn't do that, by the way"). The innovator's dilemma compounds it: a team frozen in its stack can't reinvent itself when a Temu arrives with better tech and a better business model.

The seven habits — prioritize pragmatically, kill complexity, own the work, communicate, build microteams, deliver continuously, have fun — come with concrete machinery:

- **Tech Board** — a cross-functional board (CTO, COO, CMO, logistics, finance, marketing, plus him) meeting Thursdays at noon, applying four kill-questions: will this help us reach our goals? can we actually do this with 13 people? is it small enough? do we need this now? Ideas that fail go to a "Someday/maybe" column that is "officially revisited every three months. In reality, we never do."
- **70/20/10** — 70% escaping the legacy mess toward the mission, 20% small features on production systems (capped at two-to-five days, governed by department "number twos"), 10% keeping the lights on.
- **Rule removal** — no Scrum ("I fucking hate Scrum"), no sprints, retros, stories, estimates, product owner, or pull requests; office at most one day a week. Dee Hock's principle: simple purpose and principles produce intelligent behavior; complex rules produce stupid behavior. Everyone becomes a "product engineer" who talks to the business directly.
- **Microteams + continuous deployment** — anyone picks anything off the boards, small groups self-form, ship, disband; 40–50 releases a day on a 13-person team.
- **Knowledge sharing as the work** — event storming, pairing, mobs, weekly lean coffee. Hoarding knowledge to stay indispensable is disqualifying.

## Key quotes

> "It's never the tech... it's always the organization."

The talk's spine. Everything else — the boards, the budget split, the habit list — is machinery for reallocating attention away from feature pressure and toward the mess.

> "You shouldn't ask for time for refactoring. You should just do it. It's part of your job, right?"

Refuses to make maintenance negotiable — though note the tension with his own 70/20/10 split, which is precisely a budget for it. The honest reading: don't ask permission, but do defend the allocation.

> "If you think your steps are small, make them smaller... If you deliver in three week iterations or two week sprints, that is not short, that is not small, it's still quite big."

Smallness as a recursive discipline, aimed at sprints themselves. Gall's law and Cynefin do the theoretical work: in the complex/chaotic zones, big plans never finish and estimates are fantasy; only experiments — allowed to fail — make progress.

> "We stopped using Scrum. I did that a long time ago. I fucking hate Scrum. But that doesn't mean we do Kanban... It's not Scrum or Kanban. There's like hundreds of tastes in between."

Methodology as local craft rather than menu selection. The one quasi-fixed process he describes (call it out, form, build, check in, disband) is itself "not mandatory."

> "It's your work, Joyce. You need to make your own choice."

His refusal to pick tasks for a developer who asked which system to work on. The admission that follows — autonomy can't be taught, "start with two circles... figure it out for yourself" — is the most honest sentence in the talk, and the least actionable.

> "Everything seems to look faster. That doesn't mean you go faster, because we still solve the problems... It's not yet the AI that solves the problems."

The one AI caveat in a talk that otherwise dodges the subject (he cut his prepared AI section entirely). Generation speed is not delivery speed; the hard part — the problems — is unchanged.

> "Take your mom out to dinner more. Because before you know it, you cannot do that anymore."

The closing line; his mother died a month before the talk. A process talk that ends on mortality, and earns it — the campsite-in-France story (his mother returned to the same spot for 25 years, "and that's fine") is how he frames the people who don't want change.

## Key themes

#concept technical death as an organizational disease · #concept small steps and failed experiments in complex domains (Cynefin, Gall's law) · #concept autonomy through rule removal (Dee Hock) · #pattern Tech Board governance with kill-questions · #pattern microteams that form and disband per item · #person Sander Hoogendoorn

## Critical take

The strongest material is the boring part: the Tech Board, the four kill-questions, and the 70/20/10 split are concrete, transferable governance — the talk's actual answer to "how does feature pressure get contained," and rare for being about subtraction (killing and shrinking ideas) rather than prioritization theater. The rule-removal habit is the weakest argued. It runs on accumulated authority: a 40-year veteran, author, and conference headliner can abolish pull requests and refuse to choose for Joyce; the mid-level engineer in a low-trust bank gets one slogan, "you ignite the change," which is inspiration, not a method. The pattern has visible seams: no PRs means the quality-gate question is live, and the "automate everything" section that would have answered it was skipped when his PowerPoint crashed — the talk's own failure puncturing its own thesis (which, to be fair, he then demonstrated by continuing without slides). No retros plus a three-day outage never gets reconciled; "no estimates" coexists with a two-to-five-day cap that is just coarser estimation; and the Someday/maybe column that pitchers believe gets quarterly review is, by his own cheerful confession, a garbage bin — a small institutionalized dishonesty that will eventually be discovered by exactly the people it is meant to manage. Success is never defined beyond release frequency.

What survives contact with the gaps is still worth stealing: the diagnosis (blame the org, not the tech), the recursive smallness rule, and the buses.

The meta-detail deserves its own note: the seven habits were generated by ChatGPT from summaries of his previous talks, and he then reverse-engineered the talk to fit. The skeleton is exactly as bland as you'd expect — the life is all in the margins he added (the outage, the garbage-bin column, his mother). That is a live demonstration of his own AI caveat, and of what AI-generated structure is for: serviceable scaffolding that still needs a human to make it worth hearing.

## Related pages

- [[In Praise of Normal Engineers]] — Hoogendoorn quotes Charity Majors directly ("the small unit of software ownership is not the individual, it's the team") from the other stage that same day; his product-engineer practice and microteams supply the ground-level operating detail her team-ownership thesis implies.
- [[The Forest and the Desert Are Parallel Universes]] — Hoogendoorn explicitly borrows Beck's framing ("if you live in the desert, experimentation is really wrong. However, I don't live in the desert. I live in the forest"); his rule-stripping only works in a forest org, which is exactly Beck's point that the same practices mean opposite things in the two universes.
- [[Shaped by Demand — The Power of Fluid Teams]] — North's demand-led self-assembling teams are the org-scale version of Hoogendoorn's microteams; both demolish stable-team orthodoxy, though North keeps explicit alignment constraints where Hoogendoorn mostly just removes rules.
- [[Reducing Risk in Projects, Increasing Resilience]] — the talk teaches Snowden's Cynefin as its justification for small steps and tolerated failure, and shares Snowden's precondition-over-outcome governance instinct — though Snowden wants explicit complexity science where Hoogendoorn offers method folklore.

See also [[Beyond Autonomous Teams — Simon Rohrer (Craft 2026)]], which argues "autonomy" is a container concept nobody actually wants — the direct counterpoint to Hoogendoorn's maximal rule removal — and [[AI Coding Tools Create More Bugs Than They Fix]] for the empirics behind his "everything seems to look faster" caveat.

---
*Sources: [[raw/032bd4b1a6345601ef3b00ef598661f1]], [[summary/032bd4b1a6345601ef3b00ef598661f1]]*
*Last updated: 2026-09-13*
