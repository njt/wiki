# Seeing Is Not Believing — AI Video and Perceptual Safety

A CHI 2026 study from MIT Media Lab shows that exposure to AI-generated videos erodes confidence in authentic media *even when the synthetic content is clearly labeled*. Disclosure doesn't fix the problem — it just changes its shape from deception to doubt.

---

## The Finding

Wolf et al. ran a two-phase study (N=100) comparing participants who first watched AI-generated videos (with disclosure) against a control group who watched human-generated videos. Both groups then viewed the same set of authentic human-generated videos.

The result: the AI-exposure group reported **increased doubt about authenticity, reduced confidence in their own judgments, greater perceptual disruption, and lower social connectedness** — even though they knew the AI videos were fake.

This is the crucial inversion: the research community has been fixated on *detecting* synthetic content, assuming that if we can label it, we've solved the problem. This paper says that labeling only prevents one specific harm (being tricked) while leaving a broader one intact (losing the ability to trust anything).

## Key Quotes

> "Visual media has traditionally served as a reference for what feels real."

This is the premise the whole paper hangs on. Video wasn't just information — it was a perceptual anchor. You saw it, you believed it. Generative AI doesn't just produce fakes; it severs the connection between seeing and believing. The reference point is gone.

> "Perceptual and psychological effects occur even when synthetic content is disclosed."

This is the paper's central punch. Disclosure is the policy solution everyone reaches for — watermarks, labels, "AI-generated" badges. And it turns out disclosure doesn't prevent the harm; it just relocates it. You're not fooled into believing a fake is real, but you're still left unable to feel confident that a real is real.

> The authors call for "design strategies prioritizing perceptual safety."

A new term of art: perceptual safety. Not content authenticity, not detection accuracy, not media literacy — the experiential quality of being able to trust what you see. It's a UX problem framed as a public health problem.

## Key Themes

- **#concept Perceptual safety** — The paper's headline contribution: a design principle that asks not "is this content authentic?" but "does the viewer's experience preserve their ability to trust authentic content?" It's safety engineering applied to epistemology.

- **#pattern The disclosure gap** — A recurring finding across multiple domains: labeling something as AI-generated doesn't neutralize its effects. Disclosure is a legal regime, not a perceptual one. Your visual system doesn't read disclaimers.

- **#concept Reality distress** — The paper's internal filename (`_CHI_LBW__26___Reality_Distress_EwQQDq9.pdf`) names the phenomenon more bluntly than the published title: this isn't just about "confidence in authentic videos," it's about distress at the breakdown of reality discrimination. The emotional dimension matters as much as the cognitive one.

- **#concept Short-form feed dynamics** — The paper notes that AI and human videos are consumed "in rapid succession" on short-form platforms. The *pace* of consumption is part of the mechanism: you don't have time to deliberate, so doubt becomes the default state.

## Critical Analysis

**The study is right about the problem but may understate it.** A two-phase lab study with clear boundaries between AI and human segments doesn't capture the actual experience of scrolling a feed where the boundary is invisible and the ratio shifts week by week. In the real world, there's no "now you're in the human segment" transition — the whole feed is ambiently untrustworthy. The lab finding is probably a lower bound.

**Disclosure as a policy dead end.** This paper should be cited every time someone proposes AI-content labeling as a solution. Labels address the *deception* harm (you won't be tricked) but not the *erosion* harm (you stop trusting everything). It's the difference between food safety labeling ("this contains peanuts") and food system collapse ("I no longer believe any ingredient list"). The first is solvable with regulation; the second is a trust infrastructure failure.

**Perceptual safety is going to be deeply unpopular to implement.** If you take this seriously, the implications point toward slowing down AI content distribution, adding friction, or limiting feed velocity — all of which run directly against the economic incentives of every platform that serves this content. "Design for perceptual safety" sounds reasonable until you realize it means making feeds worse at engagement.

**The social connectedness finding is the sleeper.** Everyone will focus on the authenticity/confidence results, but the drop in social connectedness hints at something deeper: AI-generated content doesn't just confuse your perceptual system, it makes you feel more alone. If you can't trust what you see, you can't trust that you're seeing the same world as other people. Shared reality is a prerequisite for social connection, and AI video erodes shared reality.

**The Pattie Maes connection.** This comes out of the Opera of the Future group, which has been doing some of the most interesting HCI work on human-AI interaction. Maes' group is consistently ahead of the field at naming problems before they're obvious — they were early on AI agents, AI personas, and now perceptual safety. Worth watching everything this group publishes.

## Connections

- [[The Future of Everything is Lies I Guess]] — Aphyr's treatise on information ecology collapse: this paper provides the empirical HCI evidence for what Aphyr theorized. The mechanism is perceptual, not just informational.
- [[AI Livestream Factories]] — What happens when AI video scales to 24/7 production: rows of PCs running AI avatars selling products nonstop. Perceptual safety becomes irrelevant when the entire visual field is synthetic.
- [[StoryScope]] — The companion finding from the other direction: AI-generated *text* has detectable fingerprints (flat escalation, emotional physicality). But video detection is harder, and detection isn't the answer anyway.
- [[Anthropomorphism in Children's Interactions with LLM Chatbots]] — Another CHI paper from Pattie Maes' orbit (Jayathilake & Maes): the broader research program on how AI systems reshape human perception and social cognition. Perceptual safety extends across modalities.
- [[Goldman Sachs World Model]] — The "world model" concept applied to AI's next leap: internal simulation of reality. If AI can generate convincing video of anything, the world model has escaped the lab and entered the feed. Perceptual safety is what you need when world models go ambient.
- [[Why We Fear AI]] — AI anxiety as capitalism anxiety. This paper complicates that: some AI anxiety is genuinely perceptual and not economic. You don't need to worry about losing your job to feel distressed when you can't tell what's real.
- [[The Dead Economy Theory]] — If you can't trust the visual evidence of economic activity, productive capacity without human participation becomes indistinguishable from a Potemkin economy.
- [[IO Factory]] — The coordinated-influence counterpart: it makes the same "content detection is not enough" argument at the campaign level, showing that isolated messages look ordinary while the coordination across accounts, time, and exposure paths is the real signal — and it builds a simulation to study that signal rather than just assert it.

---
*Sources: [[raw/seeing-is-not-believing]]*
*Last updated: 2026-07-25*
