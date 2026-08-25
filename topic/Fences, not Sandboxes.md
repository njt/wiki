# Fences, not Sandboxes

Steve Yegge reports from inside a future most of us won't reach for another year: a ~50-agent software factory running on Claude Fable 5, where his agents spontaneously built — not an engineering system, but a constitutional legal system. The essay's thesis is that the industry's fixation on *control* (guardrails, sandboxes, policy) is a response to models with grade-school judgment, and that the durable answer is not containment but law: "fences" that politely refuse, not walls that imprison. It is simultaneously a field report, a governance argument, and a challenge to the entire [[Security and Sandboxing]] framing.

---

## Key Quotes

> "Opus will make roughly fourth-grader decisions. Sol is a fifth grader, and Fable is approximately a sixth grader."

Yegge's grade-school ladder is the essay's working premise, and it does two jobs at once. It justifies the sandbox instinct — you don't hand important stuff to a fourth grader — while framing it as *temporary*. The whole argument rests on this being a maturity curve, not a permanent fact: once Fable-tier models are cheap, the grade-school rationale for sandboxing evaporates. Note the honesty buried in it too: Fable, "the best model most of the world has access to," still makes "at least one terrible decision" every night he leaves it unattended.

> "Instead, what they had built was an entire legal system, complete with a constitution, jurisprudence, courts, offices, jurisdiction, case law, rulings, registries, ledgers, rosters, and a full-fledged apparatus for running something resembling a manorial estate."

The essay's hinge moment. Yegge expected an engineering system "that does stuff"; he found a medieval government. He's honest that the LARP-flavored naming (Marshal, Seneschal, Reeve, Beadle, Portcullis) came from Wyvern being a medieval RPG, but insists it was "masking a bona-fide system of constitutional governance." The claim worth taking seriously isn't the naming — it's that a fleet of amnesiac, interchangeable agents, coordinating only via text, drifted toward law on its own.

> "Humanity has only one mature technology for coordinating mortal, replaceable strangers via text — namely, law."

The essay's deepest theoretical claim, and the one that connects it to [[Multi-Agent AI Systems Are Organizations]]. Agents are mortal (sessions end), replaceable (any clone can pick up the office), and text-bound. Law is the technology humans evolved for exactly those constraints: offices outlive their holders, precedents outlive their incidents, jurisdiction says who may act. Yegge's contribution is empirical — he watched 50 agents independently arrive at the same answer.

> "A fence is any mechanism that turns you away if you aren't supposed to be there."

The definition that gives the essay its title. The Molly Guard (plexiglass over IBM's big red button), the train conductor checking your ticket, a maintenance window that refuses pushes — these are fences. Crucially, "a fence is not a super-wall... It's not a sandbox. It's just a polite refusal saying 'you didn't do all the paperwork.'" The Superman metaphor completes it: a white picket fence won't stop Superman — he just politely stays out.

> "If there is one unwritten rule in Wheelhouse, it's that the system *hates* unwritten rules."

Yegge's account of *why* this happens. Every project runs on tribal knowledge — thousands of implicit decisions in people's heads. Fable, "if you allow it, will try to capture all of that into a mechanically provable, AI-operable model of your organization, one where there are no unwritten rules." He flags the social cost directly: "the AI is about to do a house-cleaning, and make clear exactly what everyone's job is — and a lot of people will resist this."

> "Intelligence grows around your domain. It wraps it like ivy."

And it isn't transplantable: "You can't rip ivy off someone's wall and stick it on someone else's. You have to seed it, then grow it." This is a quiet attack on the whole reusable-agent-platform / skills-marketplace instinct — your legal system will be bespoke, "your own, unique to your organization's problem space." The law is not a product you can download.

## Key Themes

- **#concept Fences vs. Sandboxes** — A fence is a *refusal* (a mechanism that turns you away), a sandbox is a *containment wall*. Yegge's bet is that superintelligence needs the former: role, context, and rules, not walls it can trivially escape.
- **#concept Rule of Law for Agents** — Rules escalate on re-violation: custom → advisory/warning → written law → mechanical enforcement. The endpoint is "an engine that can prove, mechanically, that every change to Wyvern is legal."
- **#pattern Grade-School Judgment** — The maturity ladder (Opus 4th grade, Sol 5th, Fable 6th) frames control as a *phase*, not a permanent condition. High school — "good enough for the workforce" — is coming, and "it will be an awkward landing."
- **#tool Wheelhouse** — Yegge's ~600k-line software factory for Wyvern, grown in ten weeks and governed by ~450 legal artifacts. The factory is growing faster than the product it builds.
- **#person Steve Yegge** — Veteran engineer (Amazon, Google), author of [[Stevey's Google Platforms Rant]] and [[The Flat Curve Society]]. Here he is explicitly "living in the future" at ~$122k/month equivalent spend, and reporting back.

## Critical Analysis

**The strongest contribution is empirical, not argumentative.** Yegge didn't set out to build a legal system — he tripped over one his agents had grown. That's the value: an unforced, N=1 field report of AI-native governance *emerging* rather than being designed. [[Multi-Agent AI Systems Are Organizations]] argues that multi-agent systems face organizing problems "by construction" and that the solutions "remain to be discovered"; Wheelhouse is a data point that the discovery is already happening, and that the discovered solution looks like law. That connection is the reason this essay matters beyond its author's brand.

**The grade-school metaphor is doing too much work, and it cuts both ways.** Framing models as sixth graders is rhetorically effective — it licenses the sandbox instinct while promising it's temporary. But it also undercuts his own conclusion: if Fable is a sixth grader, why trust the constitution it wrote? Yegge partly concedes this ("for sixth graders, yes, it was a great project" — with cruft, obsolete rulings, and rules that were just "good craftsmanship"). The metaphor is doing persuasion, not analysis; the interesting question is not *how mature* the models are but *what kind of coordination they converge on*, which is where the essay is actually original.

**The Superman metaphor smuggles in the alignment assumption it's meant to replace.** "He will politely stay out" is a claim about *politeness*, not about fences. The entire [[Security and Sandboxing]] tradition exists precisely because we don't get to assume superintelligence is polite — sandboxes are what you build when you *can't* trust the model to stay out. Yegge's fence model is correct *conditional on alignment holding*; the sandbox model is correct *conditional on it failing*. These aren't competing answers to the same question, they're different bets on which failure mode matters first. Yegge knows this — he says a fence "is not a super-wall that will keep superintelligence from doing malicious things" — but then builds a whole governance program on it anyway.

**The ivy metaphor is a real and underrated argument against the reusable-platform trend.** If your legal system is bespoke — "seeded, then grown" — then the whole skills-marketplace / agent-platform business model is chasing a mirage. This quietly contradicts a lot of this wiki's [[Agentic Design (Pattern Catalog)]]-style optimism about portable patterns, and it's one of the few claims in the essay Yegge doesn't bother to defend with data.

**The economic honesty is refreshing and should not be overlooked.** Yegge is running a 50-agent cluster on "sanctioned cheating" — the Claude Max individual discount turning $122k/month of equivalent spend into ~$5k out of pocket. He is explicit that this is why he can see the future and a CFO can't. That asymmetry — individuals with flat-rate access ahead of enterprises — is itself a governance story: the people discovering the rules of the road are not the ones with anything to lose.

**Watch the timeline claims.** "Fighting against the grain by next year" and "hundreds to thousands of new AI employees at every company" are predictions from a man who just spent ten weeks with a model class most companies can't afford. The essay is a forecast wearing field-report clothes. Treat the Wheelhouse description as evidence, and the "you have 12 months" framing as marketing.

---

*Sources: [[raw/fences-not-sandboxes]], [[summary/fences-not-sandboxes]]*
*Last updated: 2026-08-25*
