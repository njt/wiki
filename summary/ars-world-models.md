---
url: https://arstechnica.com/ai/2026/07/simulating-everything-sort-of-the-promise-and-limits-of-world-models/
title: "Simulating everything, sort of: The promise and limits of world models"
author: Samuel Axon
date_fetched: 2026-07-18
date_published: 2026-07-13
topics:
  - ai-research-and-models
---

Samuel Axon surveys the emerging field of world models — AI systems that simulate physical environments rather than work with language — through interviews with three practitioners: Vincent Sitzmann (MIT), Anastasis Germanidis (Runway), and Ben Mildenhall (World Labs). The piece frames world models as a potential off-ramp from LLM disillusionment, with billions in funding flowing to companies like World Labs, Runway, and Yann LeCun's Advanced Machine Intelligence.

There is no settled definition. Sitzmann describes a world model as anything that takes an interaction and simulates what happens next in an environment. Mildenhall emphasizes continuous, real-time spatial interaction rather than turn-based chat. The term is as much a branding label as a technical one, covering action-conditioned video generation, 3D asset creation, and robot policy evaluation.

Most current commercial efforts build on video diffusion models, using autoregressive diffusion to generate frames sequentially so users can intervene with input. This is Runway's approach with GWM-1. World Labs instead bakes video-derived scenes into explicit 3D representations (NeRFs, Gaussian splats), trading dynamism for practical export formats usable in today's creative workflows. The "bitter lesson" — that scaled compute and learned structure beat hand-crafted human abstractions — leads many researchers to avoid baking explicit physics or 3D knowledge into training, betting that useful physical understanding will emerge from predicting pixels.

Key open questions: whether video-based models can produce simulations faithful enough to train robots; what interfaces and representations will win (generalized vs. fragmented models); and whether autoregressive generation is economically viable given its immense compute cost. None of this is decided — the author calls it a bet, with the potential upside justifying the investment even as the field remains early and its promises unproven.
