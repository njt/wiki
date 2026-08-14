---
url: https://typesanitizer.com/blog/code-review.html
title: "Reviewing code is a skill"
author: unknown (unnamed in the post)
date_fetched: 2026-08-14
date_published: 2026 (undated)
site: typesanitizer.com
---

# Reviewing code is a skill

**Author:** Unnamed — a former physics PhD student who dropped out in 2019 to become a software engineer.

**Site:** typesanitizer.com, undated (references events up to June 2026).

## Précis

A practitioner essay arguing that reviewing code is a *skill* — learnable, teachable, and worth deliberately investing in — set against the 2025–2026 zeitgeist that treats human review as a bottleneck about to be displaced by LLMs. The author grounds the thesis in three bugs he caught in colleagues' PRs over a few weeks of work: a `git config` file-locking race, an `aws` CLI `--progress-seconds` version incompatibility, and an S3 checksum/tarball ordering failure that could have caused an outage. High-end LLM reviewers (circa June 2026) were run on all three PRs and caught none of them.

The post leans on the code-review research literature: Google's four themes (education, maintaining norms, gatekeeping, accident prevention) and Bacchelli & Bird's 2013 finding that review is less about defect detection than about knowledge transfer, team awareness, and understanding the code. It then proposes four experimental practices — randomized Socratic review dialogues, lightweight near-miss post-mortems, "firewalled modeling" (a modeler builds a formal model without looking at the code), and studying expert reviewers via Applied Cognitive Task Analysis.

The conclusion answers the "LLMs will just get better faster" objection in three moves: dropping to a lower abstraction level than your peers is always an advantage; software is young enough that the human skill ceiling is unknown; and grounding in experience reports beats social-media takes. It closes with the Yaksha–Yudhishtira exchange on skill and knowledge.
