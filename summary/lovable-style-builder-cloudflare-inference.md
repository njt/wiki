---
url: https://mainbrella.com/blog/lovable-style-builder-cloudflare-inference/
title: "Lovable style Builder via CF Inference GLM-5.3, Kimi K2.7 Code, and GLM-5.3 Flash"
author: Mainbrella
date_fetched: 2026-10-11
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - local-and-open-source-inference
---

Mainbrella Build is a Lovable-style app builder — describe an app, watch a preview, keep talking to refine it — that runs GLM-5.3, Kimi K2.7 Code, and GLM-5.3 Flash through Cloudflare Workers AI (`env.AI.run`), with Cloudflare Workflows coordinating build steps in a Linux guest. The piece is an engineering write-up of the parts around the model, not the model itself.

Two design decisions carry most of the argument. First, verification: the platform never trusts the model's "done" — it runs `tsc --noEmit` and `vite build` directly inside the workspace (so a generated `package.json` can't fake success) and feeds compiler errors back into the conversation. An operation journal distinguishes a real compiler exit code from a truncated model response or a disconnected stream, and never executes partial tool arguments.

Second, Git history without a GitHub account: source objects are stored by SHA-256 hash in R2, with D1 tracking versions and the head, and downloadable Git bundles exported from the Worker without starting a paid machine. The interesting implementation wrinkle is that incremental Git bundles can't be safely concatenated when restored objects reappear with different delta representations, so they disabled delta packing to keep exports self-contained. The rest covers launching from an existing GitHub repo (with a read-only Import app that never hands the guest an access token), an ad-hoc vs. production lifecycle with five-minute compute reservations and runtime-enforced deadlines, stable project endpoints that survive app replacement, and the honest limit that recovery starts from image and startup command — unsaved files and RAM don't come back.
