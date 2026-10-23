# It Sounds Easy

Dan Luu's "It Sounds Easy" is an argument against the most durable form of tech-industry armchair expertise: the claim that some large company's product could be built by a handful of people, and that its thousands of engineers are proof of bloat. Luu's falsifiable claim is that headcount at profitable large companies is roughly what marginal-revenue math predicts, not what outsiders' estimates of problem difficulty predict — and that outsiders systematically misestimate difficulty because they only see the happy path.

---

## The argument in one paragraph

The claim: for any large, profitable software company, the number of engineers it employs is close to the economically rational number, because businesses keep adding engineers (and salespeople) until the marginal hire stops paying for herself — and the work available for those hires is vastly larger than outsiders estimate, because the "non-core" work (optimization, i18n, security, billing, sales, research) is more work than the core service itself. This matters now more than usual because the same "sounds easy" reasoning is being applied to AI: if a weekend MVP gets you 80% of a product, the temptation to conclude the remaining 20% is bloat has never been stronger. Luu's argument predicts that any AI-era company that survives its growth phase will converge on the same large platforms orgs as its predecessors, or leave billions on the table — a prediction that can be checked against companies being built today.

## Key quotes

> Distributed systems are considered hard because real systems might drop something like 0.1% of messages, corrupt an even smaller percentage of messages, and see latencies in the microsecond to millisecond range. When I talk to higher-ups and compare what they think they're saying to what my coworkers think they're saying, I find that the rate of lost messages is well over 50%, every message gets corrupted, and latency can be months or years.

The essay's best move: quantifying organizational dysfunction with the same vocabulary we use for distributed systems, and showing that human channels are orders of magnitude worse than the networks we call unreliable. This reframes "inefficiency" from waste to be eliminated into a physical property to be engineered around.

> Businesses should keep adding engineers to work on optimization until the cost of adding an engineer equals the revenue gain plus the cost savings at the margin.

This is the economic engine of the whole argument. Headcount isn't a symptom of dysfunction; it's the equilibrium of a rational hiring process. Any argument that a company "should" be smaller has to identify where this marginal math is wrong, which almost no "could build it in a weekend" commenter attempts.

> A company that could eliminate organizational inefficiency would be a larger innovation than any tech startup, ever.

Luu preempts the standard rebuttal ("but companies *are* wasteful!") by conceding it and raising the stakes: yes, inefficiency exists, and eliminating it would be worth more than PageRank. The implication is that inefficiency is not a solvable defect but a frontier everyone operates against.

> The same thing is true for large services -- all of this stuff that people don't think of as "real" work is more work than the core service.

The closing analogy to error handling versus the happy path is the essay's load-bearing structure. It explains *why* outsiders underestimate: they evaluate the demo, not the error paths, the locales, the billing edge cases. The invisible work is the majority of the work.

> I found that it was trivial to get orders of magnitude better performance than reported in one paper and when I tried to track down the underlying source for the comments by people who work on a commercial search engine, I found that there was no experimental evidence underlying the comments, so I removed the example.

The correction note is easy to skim past but is arguably the most instructive paragraph. Luu retracted his own quantitative example because he couldn't reproduce it — a live demonstration of the epistemic standard the rest of the essay asks of commenters.

## Critical analysis

What's non-obvious here is the direction of the argument. The familiar version of this essay is anti-corporate: big companies are bloated, small teams win. Luu accepts the small-team premise (a PageRank-class innovation *could* be built by a few people) and attacks the conclusion (that the resulting company stays small). The distinction between innovation and operation is where most "why is this company so big" takes die, and Luu drives through it cleanly with the billing example — the "obvious" outsource-to-Stripe move leaves single- or double-digit conversion improvements on the table, which at scale pays for a fifty-person team many times over.

What's weak: the essay is stream-of-consciousness by the author's own admission, and it shows. The Lucene-versus-Google framing leans on a years-old blog-comment argument whose specifics have aged; the latency-revenue research is gestured at rather than cited; and the marginal-engineer argument, while economically sound in principle, is asserted rather than measured — no company is actually shown to be at its equilibrium headcount. The claim that orgs lose "over 50%" of messages is anecdote dressed as number. A skeptic can accept the structure of the argument while doubting that *every* large company is near optimal; Luu's framework explains big headcounts but also conveniently explains away any headcount, which is a weakness in a falsifiable claim.

What's left out: the demand side. If marginal engineers pay for themselves, why do the same companies run layoffs and then re-hire? Organizational politics, empire-building, and coordination failures that push headcount *above* equilibrium get one concessive sentence. The essay also predates the AI question it now speaks to: it never asks what happens to the marginal-engineer math when the marginal engineer is an agent. That's the live version of his argument, and it's unwritten.

## Related

- [[Dan Luu on AI Coding]] — Same author, same epistemic spine: where "It Sounds Easy" argues that invisible work dominates visible work, the AI-coding piece argues that invisible *testing* methodology dominates model choice, and both treat unreproduced benchmark claims as grounds for retraction rather than citation.
- [[YAGNI]] — Fowler's economic case is that building capability ahead of need loses money; Luu supplies the boundary condition, showing that past the MVP the marginal math flips and rational firms pour engineers into optimization and features indefinitely — the two essays are the two halves of one cost curve.
- [[In Praise of Normal Engineers]] — Charity Majors' claim that the org's floor, not its ceiling, is what to optimize is the organizational corollary of Luu's point that human-channel latency and message loss are the real constraint; both locate the leverage in systems that let ordinary contributors move the business.
- [[Lessons from Building Vercel v0 and the d0 Agent]] — Malte Ubl's provocation that building agents is "extremely easy" is exactly the rhetorical pattern Luu's essay names; Luu complicates Ubl by predicting that any agent-era winner past hypergrowth converges on the same multi-thousand-person platforms org, easy demo notwithstanding.

---
*Sources: [[raw/sounds-easy]], [[summary/sounds-easy]]*
