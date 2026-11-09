---
url: https://www.bobbytables.io/p/alphamaxxing
title: "Alphamaxxing"
author: bobbytables (Cadenya founder)
date_fetched: 2026-11-09
date_published: unknown
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

A first-person field report on starting a new project (Cadenya) in the agent era. The author's central claim: **"AI is not only building our software, it's also becoming the predominant user of our software"** — so the programmatic interface, not the browser UI, is now the product, and a small TAM loss comes from agents that can't use a lackluster API.

The workflow that follows is deliberately staged. First, define the API surface before any implementation: Protocol Buffers and gRPC as the IDL, with documentation and validation rules, so an agent could "take the wheel." A Claude Code session produced a first implementation that was "bad at best and atrocious at worst" — which didn't matter, because a working artifact let the author feel the API design via grpcurl and watch how Claude-the-user stumbled through it. Second, sacrificial code: he deleted the Go implementation (`rm -rf`) and rebuilt the same contract in Ruby on Rails, an experiment that influenced real decisions — ULIDs over UUIDv7, splitting Workspace from Account, machine-friendly user identities.

Both prototypes "tasted good but felt icky," so the final move was counterintuitive: write the production API implementation **by hand** in Go over three months. The payoff is framed entirely in agent terms — hand-rolling the foundations made him "Google Maps for coding agents," prompts rooted in examples rather than "now make it do this," and hours-not-days feature shipping afterwards. He closes with the lead-singer metaphor: the human sets the tempo, the agents keep clapping on their own.
