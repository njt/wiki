# Automatic Programming

antirez (Salvatore Sanfilippo, creator of Redis) draws a bright line between "automatic programming" and "vibe coding." The distinction isn't about the tools — it's about who's driving. In automatic programming, the human provides continuous vision, design judgment, and steering; the LLM is an assistant. In vibe coding, the human describes outcomes vaguely and lets the LLM produce whatever first design comes to mind, then reports failures. The punchline: "Programming is now automatic, vision is not (yet)."

---

## Key Quotes

> "Automatic programming produces vastly different results with the same LLMs depending on the human that is guiding the process."

The entire argument in one sentence. Same models, different humans, radically different output. This is the empirical foundation that separates automatic programming from vibe coding — and it's what makes the "AI will replace programmers" narrative miss the point. The programmer doesn't disappear; the programmer becomes the differentiator.

> Vibe coding is "the process of generating software using AI without being part of the process at all."

antirez's definition is more damning than most. Not "using AI loosely" or "prompting without review" — but being *absent from the process*. The vibe coder's only role is noticing when the output fails to match expectations. Everything between prompt and result is a black box.

> "Remember: it is the software *you* are producing."

He's pushing back against the learned helplessness that can accompany AI tools. When you're genuinely engaged — designing, steering, making decisions at every level — the output is yours. The AI didn't make it; you made it with AI. This isn't pride; it's an assertion of authorship and responsibility.

> "I'm a programmer, and I use automatic programming. The code I generate in this way is mine."

A declaration of identity. He's not a "prompt engineer" or an "AI operator." He's a programmer, and automatic programming is how programming works now. The term is deliberate: "automatic" modifies "programming," it doesn't replace it.

> "Redis, in its first versions, was technically unremarkable... It became very valuable because of the ideas and visions it contained."

The case study that proves the thesis. Basic data structures, basic networking — any competent programmer could have written the code. But *which data structures*, *which API*, *which tradeoffs* — that's vision. And vision is what the AI can't supply. The most successful infrastructure software of the past 15 years was an ideas play, not an implementation play.

> "Programming is now automatic, vision is not (yet)."

The closer. The "(yet)" is loaded — it acknowledges the possibility that vision itself might become automatic, but not today. Today, vision is the scarce resource. Today, vision is what you bring.

---

## Key Themes

- **#concept Automatic Programming**: AI-assisted programming where the human provides continuous vision, design, and steering. The AI is an assistant, not the driver. Distinct from vibe coding in that the human is "part of the process."
- **#concept Vibe Coding (antirez's definition)**: Generating software without being part of the process. Vaguely describe outcomes, let the LLM produce the first design, report failures. antirez is less generous than the standard definition — for him it's near-total abdication.
- **#concept Vision as Scarce Resource**: Implementation is now cheap. Knowing *what to build* and *how it should work* — the design decisions, the tradeoffs, the architecture — is not. This converges with [[Radical Accountability]] (taste is all that's left) and [[AI Zealotry]] (high-level thinking is the differentiator).
- **#pattern Ideas Over Implementation**: Redis as proof that technically unremarkable code can become world-changing infrastructure when the ideas are right. antirez is essentially saying: the code never mattered. The vision did.
- **#person antirez (Salvatore Sanfilippo)**: Creator of Redis. One of the most influential systems programmers of the past two decades. His endorsement of AI-assisted programming carries weight because he built arguably the most successful infrastructure software of the open-source era, and he's now saying the implementation was the trivial part.

---

## Critical Analysis

**This is the single best rebuttal to vibe-coding-as-identity that I've read.** antirez doesn't dismiss vibe coding — he acknowledges it democratizes software production. But he refuses to let it define AI-assisted programming. By naming his practice "automatic programming" and insisting on his identity as *a programmer*, he reclaims the frame. This is a power move dressed as a terminology post.

**The Redis case study is devastatingly effective.** It's one thing to argue that ideas matter more than code. It's another to say "I built Redis, and the code was the easy part." antirez has the credibility to make this claim, and he deploys it with the minimalism that defines his engineering philosophy. One paragraph, no self-aggrandizement, just a statement of fact.

**The "(yet)" in the final line does a lot of work.** By acknowledging that vision might eventually become automatic, antirez avoids the trap of declaring a permanent human monopoly. He's not saying AI will *never* supply vision — he's saying it doesn't *now*, and the gap between now and then is where the work lives. This is more honest than most "AI won't replace X" arguments.

**Where this connects to the broader wiki conversation:** antirez is describing [[Agent Coding Workflow]] at the "compound engineering" end of the spectrum — human-in-the-loop, vision-driven, AI as tool. His framing complements [[AI Zealotry]] (implementation is free, thinking is scarce), [[Radical Accountability]] (taste is all that's left), and [[Slowing the Fuck Down]] (deliberate friction preserves judgment). It directly contradicts the [[acceleration-flow]] dopamine-loop model by insisting on continuous human engagement.

**What's missing: the HOW.** antirez tells you *that* vision matters and *that* the human must steer, but he doesn't describe his actual workflow. What does "continuous steering" look like in practice? How does he maintain engagement across a long session? The post is manifesto, not manual. For practice, you'd need [[Addy Osmani's Workflow]] or [[TextForge Case Study]] or [[How Boris Uses Claude Code]].

**The term "automatic programming" won't catch on.** It already means something else in CS (program synthesis, code generation from specifications). But antirez probably doesn't care. The point isn't to coin a term; the point is to stake a position: I am a programmer, I use AI, and the output is mine. The term is the vehicle for the identity claim.

**See also:** [[Vibe Coding and the Maker Movement]] (scenius, evaluative anesthesia), [[Creative Firewall]] (the boundary between human and AI contribution), [[The Plan Is the Program]] (the plan as the atomic unit of work), [[Code Field]] (resist over-specification; let code emerge), [[Coding Agents and Complexity Budgets]] (agents need grep, not GUIs).

---

*Sources: [[summary/antirez-automatic-programming]]*
*Last updated: 2026-05-15*
