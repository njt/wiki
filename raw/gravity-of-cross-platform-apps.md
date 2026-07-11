---
url: https://allenpike.com/2021/gravity-of-cross-platform-apps/
title: The Persistent Gravity of Cross Platform
author: Allen Pike
date_fetched: 2026-07-11
date_published: 2021-09-01
---

# The Persistent Gravity of Cross Platform

Allen Pike runs Steamclock, a development shop that has built "dozens of nice native apps" with small teams. He has spent over a decade consulting with hundreds of companies on app development strategy.

The piece examines why well-funded companies increasingly choose cross-platform frameworks like Electron over native development, arguing that the simplistic "native = better UX, cross-platform = cheaper" framing misses the real dynamics. Pike contends that coordination costs at scale are the decisive factor.

## Key Arguments

**The primary tradeoff:** Pike states that "cross-platform UI technologies prioritize coordinated featurefulness over polished simplicity." Enterprise software in particular gravitates toward cross-platform tools because buyers value feature checklists over delight, and 75% quality is often sufficient for internal tools.

**The quadratic cost of coordination:** As product organizations grow, maintaining consistency across multiple native codebases becomes exponentially harder. Pike notes that "inconsistencies both small and large had crept into our apps over time," quoting 1Password's Michael Fey. Siloed platform teams struggle to reason about their own product, leading to slower iteration, documentation errors, and cross-team friction.

**The velocity problem:** "Slow is a dangerous place for a product company to be," Pike writes. Slow teams get outcompeted. He points to Figma and Slack as examples — products that don't feel fully native but succeeded because they "outbuilt and outcompeted their native competitors."

**The non-linear tradeoff:** Rather than a simple good-vs-cheap equation, Pike describes a curve where "the teams trying to coordinate the most feature work across the most platforms feel an incredible gravity towards cross-platform tools." Even teams that prioritize UX can find native development counterproductive when coordination overhead undermines the experience itself.

## Noteworthy Details

- Pike notes that Dropbox and Slack have both written about moving *away* from shared cross-platform core libraries for mobile, opting for fully native iOS and Android implementations instead — illustrating that the decision is never permanent.
- The 2026 addendum addresses agentic coding, arguing that AI tools have actually "strengthen the argument for cross-platform desktop apps" because when all four platform implementations are rapidly iterated by AI, human verification of quality becomes the bottleneck.

## 2026 Addendum (July 10, 2026)

In an addendum written nearly five years after the original post, Pike revisits the thesis in light of AI coding tools. His argument: agentic coding practices substantially *strengthen* the argument for cross-platform desktop apps. When AI can rapidly generate implementations for all platforms, the bottleneck shifts from "can we build it for every platform?" to "can we verify quality across every platform?" Having one cross-platform codebase means one verification surface rather than four — and human verification, not generation, is now the scarce resource.
