# Continuous Diffusion Language Models

Sander Dieleman's insider survey of the five-year ebb and flow of continuous diffusion for language: how the approach rose in 2022, went extinct after 2023 in favor of discrete diffusion, and came roaring back in 2026 on the strength of a single technical advantage — distillability. Written by someone who co-authored two of the founding CDLM papers (SED, CDCD), it doubles as a memoir of watching a field abandon an idea the author still believed in, and a technical manual for the four ingredients (embedding, loss, noise schedule, self-conditioning) any continuous diffusion language model needs.

---

## Key Quotes

> "It is difficult to say for certain why this happened, but I can think of a few potential factors: one is the ChatGPT moment, which gradually shifted the focus of language diffusion research from theoretical advantages and elegance to raw performance."

Dieleman's theory of the late-2023 "continuous extinction." Note how candidly he concedes it is speculative — he offers the ChatGPT-moment shift and Plaid-1B's **64×** training-inefficiency gap as explanations, then openly invites dissent. This is a history written by a participant who lost the argument and is trying to understand why, which makes it more honest and more suspect in equal measure.

> "This makes step distillation challenging, and overcoming that problem might require introducing significant additional complexity. Continuous methods side-step this issue completely… I think it is fair to say that few-step sampling comes much more naturally to the continuous setting. This **distillability advantage** is probably the main reason why CDLMs are back with a vengeance today."

The load-bearing claim of the whole essay. The mechanism: discrete diffusion assumes simultaneously sampled tokens are conditionally independent, so in the single-step limit it cannot model correlations at all. Continuous diffusion has no such wall — a single-step flow-map model can, in theory, capture every correlation. This is the technical reason the economics of [[The Tokens You Can't Wait For]] (text diffusion as the fix for the autoregressive decode bottleneck) actually work.

> "It is fair to assume that it probably biases the modelled distribution in hard-to-understand ways, but everyone uses it anyway, because it makes such a huge difference to sample quality that it would be an act of self-sabotage not to."

On self-conditioning — passing the denoiser's previous prediction into the next step. The striking part is the candor: this trick breaks the statelessness assumption behind ODE/SDE sampling, yet it is universal because the empirical payoff is enormous. Dieleman flags Yoo et al.'s reanalysis (self-conditioning ≈ a flat-loop approximation of a fixed-point model) as the first satisfying explanation of why the statefulness doesn't bite.

> "Because the evaluation methodology is not currently standardised, different papers use different approaches and results can be unintentionally misleading."

The essay's most important warning, and the one that cuts against its own enthusiasm. Diffusion models can trade quality against diversity at sampling time; if papers don't quantify where they sit on that trade-off, one model can *appear* dramatically better than another while merely representing a different operating point. Franca & Tong showed generative perplexity (GenPPL) — the surrogate-AR-metric most CDLM papers report — is trivially gameable. Dieleman's own "back with a vengeance" conclusion should be read with this caveat attached.

> "In a multimodal context, even the discrete/continuous divide is a distraction."

The "spicy tweet" he embeds as a one-line rebuttal to the "continuous everywhere makes multimodal integration easier" argument. His point: the hard problem isn't bridging generation paradigms, it's the *semantic gap* between abstract language tokens and low-level perceptual latents — and unified continuous diffusion does nothing to close that.

---

## Key Themes

- #concept **CDLM vs. DDLM** — Continuous diffusion language models lift Gaussian corruption into an embedding space; discrete diffusion (masked or uniform-state) corrupts tokens directly. The essay's whole arc is the oscillation between these two, from 2021 discrete → 2022 continuous → 2023 extinction → 2025 hybrid → 2026 continuous comeback.
- #concept **distillability advantage** — The reason continuous methods returned. Few-step/single-step sampling is natural for continuous diffusion; discrete diffusion's conditional-independence assumption makes step distillation hit a wall.
- #concept **self-conditioning** — Feeding the denoiser's previous prediction back as input. Ubiquitous in CDLMs despite breaking ODE/SDE statelessness; a huge empirical quality boost whose mechanism was only recently explained (fixed-point flow reanalysis).
- #pattern **the four ingredients** — Embedding strategy (explicit/pretrained/jointly-learned), loss (MSE/cross-entropy/likelihood-bound), noise schedule (adaptive, entropy-linearising), and self-conditioning. Dieleman's reusable recipe card for the field.
- #concept **flow maps** — The 2025 framework (integrate the diffusion denoiser into a one-shot map) that catalyzed the 2026 categorical extension and the subsequent "Cambrian explosion."
- #concept **evaluation methodology gap** — Quality/diversity trade-offs are unquantified and GenPPL is gameable, so "CDLM rivals DDLM" claims are currently underdetermined.
- #person **Sander Dieleman** — Google DeepMind researcher, co-author of SED and CDCD, author of the flow-maps and latent-diffusion posts this one updates.

## Critical Analysis

**The essay's best feature is its honesty about bias.** Dieleman tells you up front that he worked on the losing side, was "wistful" about its extinction, and is writing a "fairly subjective account." That self-awareness makes the history trustworthy in a way a disinterested survey wouldn't be — he flags which claims are speculation, which are "highly speculative," and where he disagrees with the community's prior consensus. The countervailing risk is that the essay is still *written by the person who lost*, and the triumphant "back with a vengeance" framing may overweight a 2026 paper flurry that hasn't yet converged. He even hints at this himself: "no clear convergence on a particular recipe."

**The distillability thesis is the real contribution, and it's testable.** The claim that discrete diffusion cannot capture token correlations in the single-step limit follows from its conditional-independence assumption — that's a structural argument, not an empirical hope. If flow-map language models scale (and he says efforts are underway), this is the mechanism that would let them compete with [[Theoretical LLM Inference Bottlenecks]]'s sequential-dependency ceiling in a way speculative decoding can only nibble at. But the essay is honest that "in practice, the limited capacity of the models still makes this challenging to do in just one step" — the theory is clean, the engineering isn't there yet.

**The evaluation caveat is the sharpest part and deserves more weight than the essay gives it.** Dieleman buries the GenPPL-is-gameable point in the "What's next?" section, but it's the single most important constraint on every claim in the piece. If the metric by which 2026 CDLMs are declared competitive with DDLMs is the same surrogate-AR GenPPL that Franca & Tong showed can be gamed, then "Continuous Diffusion Rivals Discrete" is a headline awaiting a metric fix. The essay's closing recommendation — standardize the quality/diversity frontier — is really an admission that the field's scoreboard isn't working.

**What it complicates for this wiki:** [[The Tokens You Can't Wait For]] treats "text diffusion" as one thing, but Dieleman's survey splits it — the enterprise economics of that page rest on the *continuous* variant's distillability, not on discrete diffusion, which inherits the same batching-vs-latency problems in a different form. [[Recent Developments in LLM Architectures]] is about surgical tweaks that all still presume next-token prediction; Dieleman's essay is the reminder that the paradigm itself is up for grabs, and the transformer-block optimizations Raschka surveys may be optimizing a generation strategy diffusion could partially displace. The multimodal aside also cuts against the common assumption that a unified continuous model is the natural end-state — the semantic gap between token and pixel representations is the harder problem, and "continuous everywhere" doesn't touch it. [[AuK]] is a working rebuttal-by-construction: it keeps the text encoder discrete (a frozen Qwen2.5-Omni) and applies continuous flow matching only to the audio latents, bridging the semantic gap with a learned layer-fusion over the LLM's hidden states — and its AuK-Flash release shows the distillability advantage in production, collapsing 32 ODE steps to a fixed 4-step DMD student with CFG off.

**Who should read this:** anyone who needs the map of diffusion-language-model research before the 2026 wave of "CDLM beats DDLM" papers arrives — and especially anyone reading those papers' GenPPL numbers.

---

*Sources: [[raw/continuous-dlms-html]], [[summary/continuous-dlms-html]]*
*Last updated: 2026-09-04*
