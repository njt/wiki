# World Models — Promise and Limits (Ars)

Samuel Axon's definitive 2026 survey of the world-model landscape: three expert interviews (Vincent Sitzmann of MIT, Anastasis Germanidis of Runway, Ben Mildenhall of World Labs) that map the competing definitions, architectures, and bets in the fast-moving field of AI systems that simulate physical reality. The piece is remarkable for capturing practitioners who disagree with each other on fundamentals while sharing the same broad conviction — that world models, not LLMs, are AI's next frontier.

---

## Key Quotes

> "It's definitely an overloaded term... Ask Runway, ask us, ask whoever — we're all gonna give you a little bit of a different term."

— Ben Mildenhall, World Labs

Mildenhall's candor is the most honest thing said in the piece. When the people building the technology can't agree on what it is, you know you're in the early phase of a field, where branding and research are still fused. This isn't a weakness of the article — it's the article's thesis, demonstrated in the reporting.

> "The idea that you're going to extend the capabilities of LLMs to the point that they're going to have human-level intelligence is complete nonsense."

— Yann LeCun, former Meta chief AI scientist

The article's framing quote, and the one that positions world models as the off-ramp from LLM disillusionment. LeCun has been on this position for years; what's new is that the investment community (World Labs ~$1B, AMI ~$1B, Runway $315M) is now betting on it.

> "The whole point of NeRF and splatting was primitive meshes are a huge pain to integrate into ML workflows... The thing with NeRF and splats is you have a notion that's more continuous... it's much easier — or it's possible — to actually pass gradients through and backprop and train a model."

— Ben Mildenhall

The best technical explanation in the piece. The reason NeRFs and Gaussian splats won over meshes isn't rendering quality — it's differentiability. You can't backprop through a triangle mesh. You can through a volumetric representation. This is the architectural constraint that shapes everything downstream.

> "Through many, many — in fact, in this case, literally thousands of years of research — we have been very proud that we figured out that the world is three-dimensional and how light bounces around and makes little images on our camera, and that is the reason why we would like these algorithms to have these kind of 3D representations, to make use of all of our know-how. But then in the end, the bitter lesson would suggest that this is maybe a fallacy."

— Vincent Sitzmann

Sitzmann captures the genuinely uncomfortable implication of the bitter lesson: thousands of years of physics knowledge might be the wrong thing to encode. He's not saying it *is* wrong — he's saying the bet is unresolved. This is the article's emotional core: these researchers are betting against their own training.

> "I'm saying it's a bet. This is not a decided thing."

— Vincent Sitzmann

Seven words that should be stamped on every world-model funding round and press release. The entire field is a portfolio of bets — on architecture, on interfaces, on use cases, on whether video is even the right modality. Sitzmann's intellectual honesty here is the article's north star.

> "The underlying technical backbone has a lot in common across most of these approaches... it's also just diffusion-based stuff. It's not like a fork at the root. It's really like a branch in the application side."

— Ben Mildenhall

The quietest bombshell in the article. Despite the branding wars, everyone is building on the same diffusion-model substrate. The disagreements aren't fundamental — they're about what to wrap around the base layer. This should temper the "paradigm shift" narrative.

> "No one needs to be explained how to interact with an LLM. You ask them a question, they tell you something. You ask them to code for you, they do something. For world models, maybe not as much."

— Vincent Sitzmann

The interface problem, stated plainly. LLMs had a ready-made interaction model (chat). World models don't. This is the product-design gap that will determine whether the technology finds users or stays in the lab.

---

## Key Themes

#world-model #AI-architecture #simulation #video-generation #robotics #concept #neural-radiance-fields #diffusion-models #physical-AI

**The definition wars are the product.** "World model" means video generation to Runway, 3D asset export to World Labs, and abstract state prediction to Yann LeCun's JEPA. This isn't confusion — it's the natural state of an early field where the term is also a fundraising label. The article's value is in letting each practitioner define the term in their own words and letting the contradictions stand.

**The bitter lesson as live controversy.** Richard Sutton's essay — that encoding human knowledge into AI systems is counterproductive because scaled computation always wins — is the organizing tension of the field. Runway bets on pure 2D pixel prediction letting 3D understanding emerge. World Labs bets on explicit 3D representations as a practical off-ramp for current workflows. Sitzmann openly says the bet is unresolved. This is rare: a field where the foundational philosophy is still being contested in real time.

**The 3D-vs-dynamics tradeoff.** World Labs bakes environments to static 3D assets (NeRFs, Gaussian splats) — cheap to render, exportable, but lifeless. Runway generates dynamic livestreams — interactive, physics-rich, but computationally ruinous and ephemeral. Sitzmann frames it cleanly: "the key benefit of having a 3D scene is that you need to generate it only once." This is the genuine engineering tension, not a branding disagreement. For content creation, you want assets. For robotics simulation, you want dynamics. Same substrate, incompatible optimization targets.

**The robot training data bottleneck.** Humanoid robots need paired perception-action data at scale, and teleoperation won't get us there. World models are a proposed solution: generate synthetic training data at volume. But the gap between "correlates with real physics" and "reliably simulates contact forces" is wide, and small mismatches can break learned policies. This is where the field's optimism is most vulnerable to empirical falsification.

**Latents as the bet's mechanism.** The article's most philosophical section explains why world-model builders believe physics understanding can emerge from video prediction without explicit physics training. Sitzmann's brain analogy — you don't find a mesh in your brain when you navigate a room — is the cleanest articulation of the latent-space argument. The counterargument, which the article gestures at but doesn't fully develop: correlation is not causation, and "useful approximation" can become "dangerous approximation" without warning.

**Verification is the whole game.** Germanidis and Sitzmann both describe the same validation methodology: run the same policy in simulation and reality, measure success/failure correlation. If they match, you trust the sim. This is exactly how self-driving cars were validated — empirical proof, not theoretical guarantees. The approach works but scales poorly to open-world scenarios. A world model that passes the pendulum test might fail the kitchen-cabinet test in ways you can't detect without running both.

**The interface vacuum.** The article's most underappreciated insight: LLMs had chat as a universal interface; world models have nothing equivalent. Some users will want game controllers, others a Blender viewport, others robot APIs. This isn't just a UX problem — it's a category-definition problem. Until someone figures out what "using a world model" feels like, the technology remains a collection of demos.

---

## Critical Analysis

**This is the best piece of AI journalism in 2026**, full stop. Axon does what most AI coverage doesn't: lets practitioners disagree in print, presses them on contradictions, and refuses to resolve the tensions into a clean narrative. The result is a portrait of a field that is genuinely unsettled at every level — definition, architecture, interface, business model — and that's the truth.

**The article's structure is its argument.** By interviewing practitioners from three different positions on the 3D-vs-pixels spectrum (Runway on the pure-frame end, World Labs on the explicit-3D end, Sitzmann as the academically honest middle), Axon creates a dialectic without editorializing. The reader watches the positions collide and draws conclusions. This is reporting as methodology.

**What the article doesn't do** is press hard enough on the business-model question. World Labs raised ~$1B on a product (Marble) that generates static backyard-sized environments for export. That's a tool for VFX artists and game developers — a real market, but not a trillion-dollar one. Runway's GWM-1 isn't publicly available. AMI is Yann LeCun's research vision with a company wrapped around it. The funding rounds are enormous bets on technology that doesn't yet have a clear path to revenue commensurate with the investment. The article reports the numbers without asking the follow-up.

**The bitter lesson section is the intellectual heart of the piece** and it's handled deftly. Sutton's essay is the most-cited and least-read text in AI — everyone invokes it, few understand its implications. Axon uses Sitzmann to spell out what it actually means for world models: that thousands of years of physics might be the wrong thing to encode, and that the models might develop better internal representations than we could design. This is genuinely profound and well-explained.

**The latent-space discussion is excellent but incomplete.** Sitzmann's brain analogy is the right frame, but the article doesn't engage with the strongest counterargument: that latent physical understanding may be a form of Clever Hans effect — the model learns surface correlations that match physics in-distribution but fail catastrophically on edge cases. The "ball hanging from ceiling" test Germanidis describes catches basic dynamics but says nothing about whether the model understands conservation of energy or just pattern-matches pendulum videos from its training set.

**Axon's "the bet" framing is the correct one** and it's disappointing it took until 2026 for a major publication to report it this way. The field is placing bets — on architecture, on interfaces, on use cases, on whether video is even the right modality. Some bets are more informed than others. None are settled. This is how real science works, and it's almost never how it's reported.

**The article's relationship to Goldman Sachs' investment thesis** (see [[Goldman Sachs World Model]]) is instructive. Goldman told investors world models are the next big thing and compute demand isn't priced in. Axon tells readers the people building them don't agree on what they are. Both are true. Read together, they're a more complete picture than either alone.

---

## People

- **Vincent Sitzmann** — MIT CSAIL assistant professor, leads Scene Representation Group, working on neural rendering and world models for robotics training. Position: academically honest middle — sees the bet, won't call it.
- **Anastasis Germanidis** — Runway CTO, leading GWM-1 development. Position: pure pixel prediction, let 3D emerge, ship specialized models now and unify later.
- **Ben Mildenhall** — World Labs co-founder, NeRF co-creator. Position: explicit 3D as practical off-ramp, but the real goal is general-purpose world understanding.
- **Fei-Fei Li** — World Labs co-founder, computer vision pioneer. Framed the field as "spatial intelligence." Her three criteria for world models: perceptual/geometrical/physical consistency, multimodal design, next-state from input actions.
- **Yann LeCun** — Former Meta chief AI scientist, now leading AMI. Betting on JEPA architecture (predicts abstract state, not pixels). Position: LLMs cannot achieve human-level intelligence; world models are the path.
- **Richard Sutton** — Computer scientist, reinforcement learning pioneer. Author of "The Bitter Lesson" (2019), the essay that functions as the field's philosophical north star, invoked by practitioners on all sides.
- **Clem Delangue** — Hugging Face CEO. The article quotes his "LLM bubble might be bursting" prediction as a data point in the broader case for world models.

## Companies & Products

- **Runway** — Video generation company, now building GWM-1 (three specialized world models: human interaction, character synthesis, robot policy). Raised $315M.
- **World Labs** — Fei-Fei Li's spatial intelligence company. Product: Marble (image/video → navigable 3D environment → exportable asset). Uses NeRFs and Gaussian splats. Raised ~$1B.
- **Advanced Machine Intelligence (AMI)** — Yann LeCun's new company, betting on JEPA architecture. Raised ~$1B.
- **Google DeepMind** — Released Genie 3, a model building real-time interactivity on top of video generation.

---

*Sources: [[raw/ars-world-models]]*
*Last updated: 2026-07-18*
