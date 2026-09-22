---
url: https://github.com/awlevin/typesafe-computer-use
title: "typesafe-computer-use"
author: Aaron Levin
date_fetched: 2026-09-22
date_published: 2026-09-18
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

typesafe-computer-use is a macOS computer-use agent whose every decision is made by a small classifier rather than a frontier vision model. The `clicker` CLI takes a goal in plain English and drives the Mac one step at a time: screencapture, Apple Vision OCR restricted to the frontmost window plus the menu-bar strip, a bounded accessibility-tree walk, and a merge of the two into one numbered list of clickable items. That deterministic state goes to TypeSafe's decision model (Jev, via `typesafe-sdk`) as a single request holding three `Choice` questions — which kind of action, which item, which website — plus a fourth when the app exposes pressable off-screen controls. The screenshot is never sent to a big model for the decision. An Anthropic "writer" (Haiku 4.5 by default) is called only for free text: the string to type, a URL when the site catalog doesn't cover the goal, and — once per run, on a stronger model (Sonnet 5) that does see the capture — the final answer.

The economics are the point: ~$0.0002 per decision versus ~$0.032 for Claude Opus 5 reading the same screenshot (155x), with 0.13–0.38 s decision latency versus 5.2 s, and ~1.5 s per end-to-end step versus ~5.5 s. The README is candid about the price: every piece of reasoning the frontier model does for free has to be rebuilt as deterministic code — a date parser that turns screen dates into "in N days" offsets, a filter that drops OCR lines echoing the user's own typed goal, region hints, URL validation. Mutually exclusive action sets are a stated design law: every stall encountered during development came from two options that meant the same thing, and overlapping options always read as low confidence.

Confidence, not correctness, is what the loop runs on. Each step logs every probability the classifier returned, gates acting on a 0.4 confidence floor, stops on `done`/`none`, two consecutive no-ops, or the step limit, and then hands the final screen to the writer for an answer the classifier cannot put into words. Every run writes a replayable folder — raw and annotated captures, the exact payload sent, all probabilities, per-phase timing — so a stall can be re-driven offline with `--image`. Typed text is verified by a separate TypeSafe `Noul` check and cleared on failure; passwords are never typed at all. The architecture is a single-agent harness of ~2,400 lines of Python where one module (`macos.py`) owns all platform contact, which is what makes the Linux port (xdotool + AT-SPI, PaddleOCR) a swap of one file.
