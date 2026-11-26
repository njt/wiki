# Predicting LoRA Size for Any Dataset

Engram derives a closed-form, empirically validated theory of how many bits a LoRA adapter of rank r can store, and uses it to predict the minimum rank needed for any SFT dataset — measured not in tokens but in bits under the base model, via Shannon-style code length.

---

## What it argues

The piece starts from a deceptively simple identity: a language model is a text compressor, so the "size" of a dataset and the "capacity" of an adapter can both be measured in the same unit — bits. Measure a dataset's negative log-likelihood under the base model (using 2M-token YaRN windows if needed), and you know exactly how much information the adapter must absorb. Then predict how many bits a rank-r adapter can absorb.

The theoretical core: perturbing a weight matrix to second order, with the Hessian approximated by the Fisher information and a Kronecker factorisation, the capacity of a rank-r adapter is governed by the singular values of a whitened gradient matrix — which are random, and follow the Marchenko–Pastur law. Summing the captured eigenvalues gives a closed-form curve C(r): roughly linear at low rank (~1.3–1.4 bits per LoRA parameter), plateauing toward ~0.48 bpp at full rank, recovering Wilks' theorem in the limit.

Theory can't handle full transformers' nonlinear super-additive effects (adapting several matrices jointly lifts capacity from 1.5 to 3.3 bpp), so they measure empirically: reverse arithmetic coding to mint synthetic datasets of *exactly* N bits under the model — neural linguistic steganography with random codes — train on them, fit a log-logistic curve to extrapolate to infinite-training capacity, and read the curve. The predictions land within ~5% of the smallest rank that stores 95% of real datasets (internal emails, law-firm workspaces, Django source, GSM8K, MATH-500, enwik8).

---

## Key quotes

> "a model that assigns high probability to data can encode it efficiently. And we've known this since the 1950s!"

The whole paper rests on refusing to treat "dataset size" as a token count. Bits are the only currency in which "how big must my LoRA be?" is even a well-posed question, and Shannon settles it.

> "models forget roughly as much as they learn... A model that learns 10 KB of new information might forget 12 KB of old information."

The most quietly unsettling result. Fine-tuning isn't addition; it's exchange. This deserves far more attention than the sizing formula, because it means every per-user adapter update at Engram's scale is a controlled overwriting of the base model — with "roughly predictable" but not precisely characterised collateral.

> "the size of the LoRA required depends on how many bits it takes to encode the dataset under the base model"

An elegant reframing of a hyperparameter-tuning folklore problem ("what rank should I use?") into a measurement problem. Rank stops being a vibe and becomes a function of the data.

---

## Themes

#concept (capacity in bits, Marchenko–Pastur, Wilks' theorem) #tool (arithmetic coding / neural linguistic steganography as a measurement instrument) #pattern (synthetic-data calibration: generate data of exactly known size, then read capacity off the training curve)

---

## Critical take

This is applied information theory done the right way: the theory is falsifiable (the blue curve is checked against real training at every rank) and the empirical fallback is honest about where theory fails (super-additivity across layers is admitted unexplained). The steganographic dataset generator is the real methodological gem — it converts "how much can a model memorise" from an unmeasurable abstraction into a controlled experiment, in the tradition of measuring a channel by feeding it known input.

Two caveats the post mostly brackets. First, everything assumes training data *sampled from the base model* — pure noise memorisation, a worst case. That the predictions still hold on out-of-distribution real data is a lucky empirical fact, not a theorem; signal-plus-noise datasets could behave differently, and the authors say so only implicitly. Second, "forget roughly as much as you learn" is reported as a trend, not a mechanism — which is exactly the open problem they hand to future work.

The systems implication is the commercial point: if you're provisioning thousands of per-user adapters (Engram's business), a rank that's 10× too large is real money in load time and storage. A 5%-accurate sizing oracle is a product feature, and this post is simultaneously research and marketing for it.

---

## Related pages

This sharpens [[FLUX.2 klein LoRA Fine-Tuning]]: that page treats rank as an empirical knob to be swept for image LoRAs; here rank becomes a predictable function of dataset entropy — the same adapter form, but with the sizing question finally given a theory.

It strengthens [[Sparks of Weight-Space Programming]]'s picture of weights as a writable medium: that work reads and edits weights directly, this one quantifies exactly how many bits of that medium a low-rank write can carry.

It nuances [[Hobson — Brooker's Home-Built Jev-Class Classifier]], where LoRA was chosen pragmatically on a single 3090 with no capacity theory; Engram's formula would have told Brooker upfront whether his dataset could even fit at low rank.

And it resonates with [[They're Made Out of Weights]] on parameter-efficiency as the frontier of customisation: Engram's tradeoff survey (prefixes, steering vectors, sparse memory layers, trainable experts) reads as a menu of encodings, each with a different bits-per-parameter exchange rate.

---
*Sources: [[raw/lora-size]], [[summary/lora-size]]*
*Last updated: 2026-11-26*
