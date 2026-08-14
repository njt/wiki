# Agentic Engineering at Kenn

Wes McKinney's August 2026 account of how Kenn Software's three-person team ships hundreds of pull requests per week with a low bug rate, written as a direct rebuttal to the "loop engineering" and "graph engineering" hype flooding X and LinkedIn. The headline is "I think loops are bullshit" — but the real argument is narrower and more interesting: fully autonomous, no-human-in-the-loop pipelines are the bullshit, while **human-operator loops** — where the clankers do the typing and checking and the human stays at the design-and-taste layer — are the entire point.

---

## Key Quotes

> "I think fully autonomous, no-human-in-the-loop pipelines are bullshit: anyone who is telling you that you can engineer agents looping on each other's output, step away from the keyboard, and get good quality output on the other end, in almost all cases, is either a) clueless or b) selling you something."

McKinney's thesis, compressed. Note what he *doesn't* reject: he still runs loops, and his own diagram says "loop until it converges." The pedants catch this ("Wes, your diagram says 'loop until it converges'"), and his answer is the crux — these are *human-operator* loops. The clankers are not in charge.

> "Vibe coding is not caring at scale."

The team's blunt restatement of Jesse Vincent's "planning, architecture, and… caring about the output." This is [[The Cult of Vibe Coding Is Insane]]'s "bad software is a decision you make," said by a team running three people against millions of lines.

> "The clankers (what we call the coding agents, since 'agent' gives them too much credit) are not in charge, we are."

The neologism is the argument. "Agent" implies autonomy and judgment; "clanker" implies a tool that clanks along doing what it's told. Renaming is a discipline — it keeps the accountability where McKinney thinks it belongs.

> "The work produced by the latest frontier models (5.6-Sol and Fable) is extremely sloppy and almost never suitable for production without substantial hardening."

A rare blunt admission from someone deep in the tooling. The models are faster, not more careful — which is precisely why the verification layer (roborev) exists. On large changesets, "we sometimes spend hundreds of dollars in tokens bug-bashing."

> "Communicate for humans. Lead with outcomes, skip the blow-by-blow, and describe PRs as they exist now, with no robospeak walls of text."

One of the seven Clanker Constitution principles, and a pointed jab at the default "robospeak" behavior he attributes to Claude Fable. The constitution is a governance layer for agents who are, in McKinney's telling, "bad at communicating with humans; they are sloppy and make messes; they overstep boundaries."

---

## Key Themes

**#concept Human-Operator Loops** — The distinction McKinney actually defends. Autonomous loops (agents looping on each other's output, human absent) are "bullshit"; loops with a human operator at every design and taste decision are the workflow. It's a restatement of the whole wiki's verification-over-generation thesis, from a production operator's chair.

**#pattern The Spec-and-Adversarial-Review Pipeline** — Design together → second opinion from a different model family → Superpowers writes the spec → a separate agent reviews it adversarially until convergence → implement in small pieces → roborev verifies asynchronously → `roborev-fix` closes everything out. The spec is the control surface; the reviews are the gate.

**#tool The Kenn Stack** — Kenn Forge (review/land workspace, a local cached view of GitHub because GitHub "struggles to hold even one nine of uptime"), Ghosthub (multiplexer-native terminal), Kata (system of record for intent), AgentsView (token/session intelligence), roborev (verification). Each owns one layer; together they're an argument that the "legacy stack" can't survive agentic volume.

**#concept The Clanker Constitution** — Seven operating principles launched into every agent session: honor the request, act with judgment, finish the job, protect existing work, verify reality, communicate for humans, learn in the right place. It's an answer to the *behavior* problem (agents that overstep, make messes, talk in robospeak) that no model upgrade fixes.

**#concept "Caring at Scale"** — Vincent's "caring about the output," operationalized. The differentiator between vibe coding and agentic engineering isn't tooling, it's whether a human is still accountable for the result — "the human remains accountable for the result."

---

## Critical Analysis

**McKinney is a tool-builder talking up his own stack, and it shows.** The piece is genuinely useful process documentation, but it's also marketing — AgentsView and roborev "are great and completely indispensable: if you aren't already using them, do so immediately!" lands differently once you know roborev, Kata, and Claude Chic are all his ([[Kata]], [[Claude Chic]]). The "do so immediately" is the giveaway. That doesn't invalidate the process, but the reader should discount the enthusiasm by the self-interest.

**"Loops are bullshit" is a rhetorical feint, and he knows it.** The post opens with a punchy claim and spends the rest of it walking back to "human-operator loops are fine." It's a marketing hook — but a useful one, because it correctly names the actual scam: the *lights-off* dark factory sold as a product. He's on the same side as [[Harness Engineering is not Enough]]: maintainability can't be rewarded by a test run.

**The unstated tension with supervision fatigue.** McKinney's answer to the exhaustion [[Human-in-the-Loop is Tired]] diagnoses is *more process* — push the human up to the design/taste layer and automate the mechanical verification. That may reduce drudgery, but it relocates rather than removes the cognitive load: the human is now the only thing holding a coherent vision across hundreds of PRs a week. The clanker constitution is, from one angle, a way to make agents less exhausting to supervise; from another, it's an admission that they're exhausting to supervise.

**The $56,836/month token bill is the honest economics nobody else publishes.** [[Agent Coding Workflow]]'s "What's Missing" section flags cost modelling as thin; McKinney gives one real number — a top-of-leaderboard, subscription-subsidized number that most teams will never reach but that finally anchors the "burning tokens" discourse to a dollar figure.

**The "living architecture documents" point is underrated.** He refuses to retain Superpowers spec/plan docs in repos, converting them instead into living documents for humans and future agents. That's a claim about what's durable — the *architecture*, not the plan that produced it — and it sits cleanly alongside [[Specifications as the Product]].

---

## Connections

- [[Agent Coding Workflow]] — the hub this page slots into; McKinney's is the team-scale, production-end of the maturity spectrum
- [[Specifications as the Product]] — Superpowers' spec → adversarial-review → implement loop is the spec-as-product thesis run as daily practice
- [[Human-in-the-Loop is Tired]] — Summers names the exhaustion; McKinney's process is the tooling-side answer, and the open question is whether it relieves or relocates the fatigue
- [[The Cult of Vibe Coding Is Insane]] — "vibe coding is not caring at scale" is Cohen's "bad software is a decision you make" said by a team that ships
- [[Radical Accountability]] — McKinney's own earlier thesis (taste is the only remaining excuse) is the philosophical substrate of this process
- [[Kata]] and [[Claude Chic]] — two of the tools in this stack, both McKinney's; the article is the umbrella that explains why they exist
- [[The Mythical Agent-Month]] — the "agentic tar pit" McKinney also describes; this piece is his prescription for not falling into it
- [[Guardrails and Feedback Loops]] — roborev is the mechanical enforcement layer in action

---

*Sources: [[raw/agentic-engineering-aug-2026]], [[summary/agentic-engineering-aug-2026]]*
*Last updated: 2026-08-14*
