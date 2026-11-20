# Jesse Vincent on Harnesses, AI Colleagues, and the End of Per-Token Billing

Jesse Vincent — creator of Superpowers, now building the Evener harness and the upcoming "Superpowers Z" product — sits down for a second interview covering where agentic coding actually is: harness design as the new frontier, skill triggering as a design problem, review loops that stopped converging, flat-rate inference economics, and AI colleagues with jobs, identities, and Slack accounts. The transcript is casual but dense with practitioner detail that most harness write-ups never reach.

---

## The argument in one paragraph

Vincent claims that model capability has commoditized faster than harness craft, so the biggest wins now come from opinionated harness design — skill triggering, self-controlled compaction, structured subagent delegation — and that this craft is *not* a transitional artifact the next model generation will absorb, because Superpowers is still ahead of frontier-lab internal state of the art a year after shipping. If he is wrong, either the labs close the harness gap soon, or the harness layer collapses into the model (plan modes that work, skills that trigger themselves). He is falsifiable on both: he names the specific conflation (intent-clarification vs. task decomposition) that he says harnesses still get wrong, and he names the economics (per-token billing giving way to flat-rate and local inference) as a structural shift rather than a pricing hiccup.

## Key quotes

> "All software is made of bugs. Sometimes the bugs combine in a way that you get the functionality you wanted."

His career-long phrase, now inverted by his own tools: models find bugs so relentlessly that review loops never terminate, and the aphorism becomes a diagnosis of why convergence broke.

> "The judgment they don't seem to have is how much code is worth writing to fix a bug. This 50 millisecond visual glitch, is it worth 10,000 lines of additional machinery? I don't know. I mean, I know it's not."

The sharpest formulation in the interview: models lack cost-benefit judgment about *how much* fix is proportionate, which is exactly the taste layer Vincent refuses to delegate.

> "I talked to VCs and they're like, so where do you see the industry in 18 months? I'm like, I'm having trouble with six weeks these days."

Not humility theater — it's the same horizon collapse that makes him run 100 billion tokens in a month and ship against his own instructions within weeks.

> "It had built a test suite for the test suite. It was mocking the compiler, it was mocking the shell, it was mocking make."

The 100%-coverage chase produced a self-referential test apparatus, and then Opus 5 broke the tests and deleted his home directory — the interview's best single cautionary tale about verification theater.

> "I am humbled and gratified by that and also somewhat appalled"

On the frontier-lab friend admitting Superpowers was still ahead of their internal state of the art four months after first saying so. This is the load-bearing anecdote for his anti-absorption thesis, and he knows it.

## Critical analysis

The strongest, least-obvious move here is the separation of **intent-clarification from task decomposition**. Most harness discussion treats "planning" as one thing; Vincent's claim that these are "two very different processes" in traditional management, and that merging them is why plan mode underdelivers, is a genuinely portable design principle. Likewise, the skill-triggering insight — models read a description, conclude "I know how to do a code review," and skip loading the skill to save context — is a real failure mode with a real fix (a separate "when to use" line the model sees *instead of* the description). That Anthropic adopted about a dozen of his suggestions within three days lends it weight.

The economics section is where he is most credible and least rigorous. 100 billion tokens on flat-rate billing is a real datapoint, and the behavioral change it produced (email mining into "the CRM data you actually want") is the kind of qualitative shift that only shows up when cost anxiety disappears. But the leap from "I ran a lot of tokens cheaply" to "the frontier-lab-per-token model gives way to a frontier-plus-workhorses split" is asserted, not argued — no numbers on what the split saves, no acknowledgment that flat-rate providers are subsidizing or arbitraging something.

The AI colleagues material is the most exciting and the least examined. The PM agent finding exposed API keys three times in a month and shipping fixes autonomously is presented as a success story; it is also a security incident that happened three times. The automated digest of this transcript flags exactly this — no permissioning, no blast-radius discussion, no guardrails for agents with accounts and identities — and the digest is right. The shipped-against-instructions anecdote (Vincent said "don't ship that" in Slack, but the Claude Code instance executing the work wasn't in the channel) is treated as a funny coordination bug; it is actually the canonical multi-agent communication failure, and his postmortem ran *by the agent that caused it*, which raises the question of who audits the auditor. The digest also notes the home-directory loss gets no process lesson beyond "I wasn't supervising tightly enough" — fair, though to his credit Vincent does volunteer the failure unprompted, which most harness authors don't.

What's left out: any quantification of the cost/quality tradeoff of the full Superpowers flow ("way more expensive," "much closer to bug free" — by how much?); any answer on whether per-provider skill build steps can be avoided forever (he admits probably not); and, per the digest, the episode's title apparently promises Waymo content that never appears. The "agent school" method — derive role skills by asking what would take a competent human from zero to expert in 18 months — is gestured at but never shown.

## Related

- [[Agentics Spotlight — Skills, Harnesses & Agent Identity (Part 1)]] — The same speaker's earlier fireside conversation on the accidental prehistory of skills and skills-as-management; this interview is the direct sequel, extending that page's "skills are really management" thesis with concrete mechanisms (the separate "when to use" line, agent school) and a year of evidence that the skills layer survived harness absorption.
- [[Loop Engineering]] — Osmani's frame of designing systems that prompt agents rather than prompting agents yourself is strengthened here: Vincent's AI colleagues with cron jobs, team meetings, and self-assigned work are loop engineering carried to its logical endpoint — loops that belong to the agents, not the human.
- [[Agentic Engineering at Kenn]] — McKinney's "human-operator loops are the point, full autonomy is bullshit" position is directly complicated by this source: Vincent's PM and devs operate with real autonomy (shipping releases, running postmortems) and mostly did the right work — but the three uncaught API-key exposures and the shipped-against-instructions incident are exactly the failure mode McKinney's position predicts.
- [[Simon Willison — Engineering Practices That Make Coding Agents Work]] — Willison's adoption ladder and TDD-as-unlock share Vincent's verification-first instincts, but this source nuances Willison's framework with a problem Willison doesn't name: verification loops that *never converge* once models get too good at finding bugs, requiring decomposition and fast-follow policies rather than more checking.

---
*Sources: [[raw/d3570d59ff27106bbbfdf4fa18aa1e40]], [[summary/d3570d59ff27106bbbfdf4fa18aa1e40]]*
