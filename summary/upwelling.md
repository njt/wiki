---
title: "Upwelling"
url: https://www.inkandswitch.com/upwelling/
date_fetched: 2026-05-14
section: "Random"
topics:
  - misc
---

# Upwelling: Real-Time Collaboration with Version Control for Writers

An experimental editor from Ink & Switch combining real-time collaboration with version control concepts for professional writers.

Key problems identified:

The "Fishbowl Effect": Real-time collaboration creates stress as writers feel monitored. "Writers don't want first drafts visible to the editor." Writers resort to workarounds like offline mode or copying documents to private files.

File-Based Versioning Issues: Desktop software avoids the fishbowl effect but creates manual merge problems. Teams generate unwieldy filenames like "Temporal Paper - Final - PLOS Revisions v3.docx" as makeshift version control.

Editorial Review Challenges: Change-tracking interfaces overwhelm readers. "I hate Track Changes! I'm dyslexic. I find reading Track Changes nearly impossible."

Inadequate Change Grouping: Tools treat every keystroke sequence as a separate suggestion, cluttering interfaces.

Design Philosophy: Borrows from Git's branching and merging while avoiding technical complexity. Supports deliberate divergence and convergence through drafts, syntactic and semantic conflict recognition, meaningful change grouping resembling commits, and floating drafts that rebase atop the stack when merged.

Core concepts: Layers and Drafts (documents comprise layers; unmerged layers are "drafts"), The Stack (merged layers form linear history), Always-on Change Tracking (separating record-keeping from visualization), Floating Drafts Architecture (active drafts auto-rebase when another merges).

Key findings:
1. Always-on tracking empowers writers
2. Both real-time and async modes necessary
3. Automatic merging has limits (CRDTs handle syntactic but humans must detect semantic conflicts)
4. Prevention beats resolution (named drafts communicate intent)
5. Keystroke grouping matters
6. Independence across drafts simplifies workflows
7. Stack transparency ensures reviewers see what they approve

Built on Automerge CRDT, ProseMirror editor, React/TypeScript frontend, NodeJS backend. Works offline.
