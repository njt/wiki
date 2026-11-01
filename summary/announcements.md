---
url: https://slcr.dev/announcements#2026-10-29-slcr-launches
title: "2026-10-29: SLCR launches"
author: "The team at Wack"
date_fetched: 2026-11-02
date_published: 2026-10-29
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

Wack launched SLCR (Sliceable, Linked, Checkable Requirements, pronounced "slicer") on 2026-10-29: an open format for writing software requirements aimed at both people and coding agents. The motivation is stated plainly — their own agents kept drifting from the spec over time — and the target user is anyone on a long-lived codebase who is handing more implementation work to agents.

The pitch is verification-shaped rather than generation-shaped: SLCR gives you a way to check that what an agent produces still matches what you asked for. The launch ships three artifacts: a getting-started tutorial (install the CLI, write your first spec), an agent skill that teaches agents to read and write the format, and an open-source GitHub repo as the venue for issues and PRs.

The name telegraphs the design: requirements that can be sliced into independently checkable pieces, linked to each other and presumably to code, and checked — machine-checkable acceptance criteria rather than prose. It is a launch announcement only, so all substantive claims (does it actually stop drift? how does checking work?) remain to be demonstrated in the docs and repo.
