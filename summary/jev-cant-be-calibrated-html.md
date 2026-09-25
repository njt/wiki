---
url: https://www.alexmolas.com/2026/09/23/jev-cant-be-calibrated.html
title: "Jev can't be calibrated"
author: Alex Molas
date_fetched: 2026-09-25
date_published: 2026-09-23
topics:
  - ai-research-and-models
  - guardrails-and-feedback-loops
---

Alex Molas argues that Jev — TypeSafe's System One model that returns typed
decisions with probabilities — is genuinely useful as a zero-training-data
universal classifier, but that its headline "calibrated probabilities" claim
cannot hold in general.

The core argument is definitional: calibration is a joint property of a model
*and* a data distribution. A model can be calibrated on TypeSafe's training
distribution (even granting RLCD does what it claims) and still be miscalibrated
on yours, because two companies can share a label ("spam") while having
different base rates — yet Jev returns the same probability to both. So the
probabilities can't be trusted for thresholds, expected-cost arithmetic, or
mixing with other models without local recalibration.

He also cites evidence the failure is worse than distribution shift: Jev says a
fair coin lands heads with probability 0.92 even though the true probability is
stated in the prompt, and a recent experiment finds `Noul` much better
calibrated than `Choice` on the same problem — meaning the probabilities have
different semantics depending on which primitive you use.

His prescription: treat Jev's outputs as good *scores* (they rank examples
well), not probabilities. If your system depends on the actual number,
calibrate on your own data — a few hundred labeled examples are enough to fit
a Platt scaling on top.
