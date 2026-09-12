# Martin Fowler and Kent Beck on Reinventing Software

Two authors of the Agile Manifesto, 25 years later, grapple with an AI revolution that makes every prior technology shift look small. The conversation is honest, unsettled, and refreshingly free of answers. Fowler and Beck don't pretend to know where this is going — they offer frameworks for figuring it out.

---

## What's at stake

The core argument: AI is not just another technology wave. It's a change of magnitude and speed that nothing in Fowler or Beck's careers compares to — not objects, not the internet, not agile. And critically, **nobody has the answers**. For 25 years, experienced developers could "press play" on known solutions. That era is over. The answers change week to week, sometimes day to day. The skill that matters now is not knowing — it's knowing *how to know*.

This reframes the senior engineer's job. It's not about having the playbook anymore. It's about demonstrating how to figure things out in a world where the playbook is being rewritten in real time.

---

## Key quotes

> "People want the answer and the answer is changing. So you can't possibly, in this environment, have the answer now. That's the bad news. The good news is nobody else has the answer either. So you're just as smart as everybody else because you're just as ignorant as everybody else."
> — **Kent Beck**

Beck's most liberating insight. The playing field is leveled — not because everyone got smarter, but because the old expertise doesn't transfer. This is terrifying for people who've built careers on accumulated knowledge, and exhilarating for those who were never on the inside track. Beck follows this by saying it behooves seniors to demonstrate not just *how to use* these tools effectively, but *how to figure out* how to use them effectively. That meta-skill is what he's now trying to model.

> "My skepticism has to be absolute and total, which means I have to be skeptical about my skepticism."
> — **Martin Fowler**

Fowler's epistemological double-take. He applied it to blockchain (verdict: snake oil) and almost applied it to AI autocomplete (verdict: wrong, and he caught himself because he kept reading people who knew more than him, specifically Simon Willison). The discipline is: be maximally skeptical of every claim, but maintain *at least as much* skepticism toward your own initial reaction, because it might be wrong too. This is hard. Most people pick one pole and stay there.

> "What's the smallest experiment I can run to verify to my own satisfaction whether this claim is true? That's the skill that is suddenly in the last year become a thousand times more valuable."
> — **Kent Beck**

The practical implementation of Fowler's skepticism. Not "what does the consensus say," not "what does the benchmark show," but *what is the least I can do to satisfy myself*. The emphasis on "to my own satisfaction" is deliberate — everyone's threshold is different, and outsourcing your judgment to benchmarks or thought leaders is the mistake. This is [[Vibe Coding as a Team Sport|Kasparov's insight]] applied to technology evaluation: weak human + small experiment + honest assessment beats strong human + assumptions.

> "It turns out that people don't want faster, cheaper, better. … Inside a company, the incentives are so misaligned with actually achieving that. … People will punish you for that if that doesn't align with their incentives inside of organizations."
> — **Kent Beck**

The single most cynical and honest moment in the conversation. Beck has spent 25 years watching agile get co-opted by the Agile Industrial Complex, and he sees the same pattern accelerating with AI. Faster/cheaper/better sounds like an obvious win, but inside organizations it threatens people whose status, budget, or headcount depends on things staying the same. AI tools promising productivity gains will hit exactly the same wall that agile hit — not a technical wall, an incentive wall. This directly echoes [[Uber — Agentic Engineering Shift|Uber's unresolved measurement gap]] between activity metrics and revenue impact.

> "Instead of having 50 people on my team, I have five people on my team. They don't have to talk to each other, and they can each have 10 agents. And that's the same. It's not the same."
> — **Kent Beck**

Beck's pushback against the "re-soloing" narrative. Extreme Programming was built on the premise that software is a social activity — hours of daily conversation between people who disagree, who have different energy levels, who force each other to explain things. Replacing that with one person managing six agents isn't a more efficient version of the same thing; it's a fundamentally different activity that loses the friction that produced good results. The messy, social, chaotic process *was the point*.

> "The Venn diagram of developer experience and agent experience is a circle."
> — **Insight from the Future of Software Engineering conference, quoted by Martin Fowler**

What's good for humans is good for agents. Modular code, good tests, precise domain language — these aren't just craft virtues anymore, they're the interface through which agents become effective. Fowler is cautious about whether he's indulging in wishful thinking, but the feedback he's collecting from practitioners consistently points in this direction. This means the existing craft investment isn't wasted — it's being repriced as agent infrastructure. See also [[Agent Coding Workflow]] and [[The Oracle Is the Asset]].

> "I take a kind of OCD enjoyment in the craft and I need to let go of that because that satisfaction of getting this one function just right just doesn't make a difference anymore."
> — **Kent Beck**

The most personally affecting quote in the conversation. Beck says this *with sadness*. The deep flow state of making one function perfect — the tiny safe steps, the emerging clarity, the satisfying pop into focus — that was the thing he loved. He's not saying it's gone; he's saying it no longer has *leverage*. The new source of satisfaction is understanding the domain and its connection to the program. This is a shift from local optimization to global understanding — a different kind of craft. It connects to [[The Joy and Power of Understanding]], which makes the same argument from a different angle: understanding is both the pragmatic path and the intrinsic reward.

> "I'm very much concerned we're going to have some really bad security incidents over this year because people are just not paying attention."
> — **Martin Fowler**

Fowler is specifically alarmed by groups wanting to give LLMs full control over email — read everything, reply to most things. He calls the security risk "mind boggling." This isn't theoretical; he's hearing this from "surprisingly large companies." The blind rush to automation is outpacing security thinking by a wide margin. See [[Security and Sandboxing]] for approaches to this problem.

---

## Themes

### #pattern — Total skepticism

Fowler's "skeptical of my skepticism" is a meta-cognitive discipline, not a personality trait. It requires: (1) assuming every new claim is wrong, (2) assuming your own initial reaction might also be wrong, (3) designing the smallest possible experiment to resolve the tension, (4) listening to balanced, credible voices who've done more experiments than you. Most people stop at step 1.

### #concept — The amplifier model

Beck's circular saw metaphor: AI doesn't end carpentry, it removes drudgery and amplifies the carpenter's skill. This applies asymmetrically — juniors who are already learning fast learn faster, experienced developers who are already effective become more effective. The people in the "middle" who entered programming for money, not craft, face the biggest displacement risk. This is a more nuanced version of the "10× engineer" framing — it's not that AI makes everyone 10×, it's that it widens the gap between those who use it well and those who use it poorly.

### #concept — The re-soloing illusion

The seductive idea that one programmer + N agents = N+1 programmers. Beck argues this confuses tool use with collaboration. Real collaboration involves disagreement, different energy levels, social accountability, and the generative friction of explaining things to someone who doesn't already agree with you. An agent doesn't provide that. It provides compliance.

### #pattern — Two humans + n genies

Beck's preferred mode: pair programming where both humans interact with AI agents. The slowness of current models turns out to be a feature — the 3-minute gap while the model processes creates space for human conversation about naming, structure, and next steps. Faster models would actually make this worse by compressing that conversational space. The pattern preserves the social benefits of pairing while leveraging AI.

### #concept — DX = AgentX

The "Venn diagram is a circle" insight: practices that make developers effective (modular code, good tests, precise domain language) are exactly what make agents effective. This is a convergence argument — the craft isn't obsolete, it's being repriced. What was "nice to have" becomes "necessary for agent productivity."

### #person — Simon Willison as credibility signal

Fowler's story of almost dismissing AI is instructive. His early experience with Copilot-style autocomplete was negative — mostly garbage completions. What kept him from flipping the "bozo switch" was reading Simon Willison's blog, which provided balanced, honest, "I don't know when I don't know" coverage. Fowler's meta-point: find the Willisons in your field — the people who give you both the good and the bad and are willing to say "I don't know."

### #concept — The Agile Industrial Complex as warning

The core ideas of agile were good. What grew around them — certifications, consultancies, tooling that promised "agile in a box" — was mostly snake oil. Fowler and Beck see the same pattern accelerating with AI, at much higher speed and scale. The challenge of distinguishing real value from hype isn't new, but the stakes and velocity are.

---

## Critical analysis

**The most valuable thing here is the framing, not the answers.** Fowler and Beck are modeling a stance toward technological uncertainty that the industry desperately needs. They're not selling a methodology, a tool, or a certification. They're saying: here's how two people who've seen a lot of snake oil think about something genuinely new. The meta-skill they're demonstrating — experiment, listen to credible skeptics, hold your own certainties lightly — is the real product.

**The "DX = AgentX" convergence is the most actionable claim and the least tested.** Fowler admits he might be indulging in wishful thinking. The idea that good tests and modular code make agents more effective is intuitively appealing and has some practitioner reports backing it, but the causal arrow could point the other way: maybe the kind of teams that write good tests and modular code are also the kind of teams that use agents effectively, for reasons that have nothing to do with the code structure. This needs empirical work. If it's true, it's the most important finding in the conversation — it means the entire craft tradition retains leverage. If it's false, the craft tradition is decorative.

**The "middle" displacement problem is raised and abandoned.** Beck identifies the large cohort who entered programming for money as the group most at risk from AI amplification, then says "I don't know where the middle is going to go." Fair enough — nobody knows — but this is the question that matters most for the industry's social contract. If 30-50% of programmers are displaced, the "golden age of the junior programmer" is cold comfort for the ones who don't make it through. The conversation lacks any engagement with what retraining, transition, or safety nets might look like.

**The security blind spot is staggering.** Fowler flags that large companies are seriously discussing giving LLMs complete control over email, calls it "mind boggling," and predicts serious incidents. He's almost certainly right. But naming the problem isn't the same as addressing it, and neither speaker offers any framework for thinking about AI security beyond "pay attention." For a conversation with this much organizational wisdom, the absence of any safety engineering discussion is a gap.

**The social argument is undersold.** Beck's defense of XP's social environment is the most contrarian and important thread in the conversation, but he doesn't fully develop it. The claim that messy, social, high-friction collaboration *produced better results* is a testable hypothesis about software quality, not just a preference for human contact. If true — and XP's track record suggests it might be — then the re-soloing trend isn't just sad, it's counterproductive. Someone should study this.

**What's missing: the economic layer.** The conversation stays entirely at the level of individual developers and teams. There's no discussion of how VC funding shapes AI tool development, how concentration of AI provider power affects the ecosystem, or what open-source alternatives look like. For people who spent 25 years watching economic forces shape the agile movement, this absence is notable. The Agile Industrial Complex didn't happen because the ideas were bad — it happened because there was money to be made. The same forces are at work with AI, at much larger scale.

**The conversation accidentally demonstrates its own thesis.** The most revealing moment is when Beck describes his sadness at losing the craft high of perfecting a single function. He's describing exactly the kind of experience that made him a great programmer — and he's saying it no longer matters. If the people who built the craft tradition are saying the tradition's core pleasure has lost its leverage, that's either prophetic or premature. I suspect it's both: the pleasure of local optimization *is* losing leverage relative to the leverage of understanding the whole system, but Beck may be undervaluing how much the local craft taught him about how to achieve global understanding.

---

## Related pages

- [[Software Engineering Craft]] — Hub for craft fundamentals
- [[The Joy and Power of Understanding]] — Understanding as both pragmatic path and intrinsic reward
- [[YAGNI]] — Fowler's canonical 2015 bliki entry: the four-cost framework for presumptive features, and the enabling relationship between YAGNI and refactoring that underpins his later work
- [[The Cost YAGNI Was Never About]] — Beck's own reframing of YAGNI as options pricing
- [[Vibe Coding as a Team Sport]] — Kasparov insight applied to software: process matters more than raw capability
- [[Software Engineering at the Tipping Point]] — Adam Bender's 10× amplifier framing
- [[Writing Code vs. Shipping Code]] — 180% commit gains attenuate to 30% at release level
- [[Agent Coding Workflow]] — The practitioner's daily loop with AI agents
- [[The People Who Will Thrive in the AI Age]] — David Brooks on volition vs. intelligence
- [[Engineering for Bounded Cognition]] — Human cognitive limits as design constraint
- [[The Oracle Is the Asset]] — The test suite as durable asset, not the code
- [[Security and Sandboxing]] — The security thinking Fowler says is missing
- [[Running an AI-Native Engineering Org]] — Organizational dynamics Fowler and Beck describe from the outside
- [[The End of Code Review]] — What happens to verification when agents write code
- [[Vibe Coding and the Maker Movement]] — Evaluative anesthesia and the dopamine of making
- [[Uber — Agentic Engineering Shift]] — The measurement gap between activity and revenue impact

---

*Sources: [[summary/fowler-beck-reinventing-software]], [YouTube](https://www.youtube.com/watch?v=CZs8J1ZD0CE), [Gist transcript](https://gist.github.com/6576b007d7c3f21554435b2b2923492e)*
*Last updated: 2026-07-04*
