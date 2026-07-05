# Upwelling

An experimental editor from Ink & Switch that finally takes the best idea from software development -- branching and merging -- and makes it usable by writers. Authors can collaborate in real time when they want to, but also work on private drafts that only merge when they're ready. It addresses the fundamental tension between creative privacy and collaborative editing.

---

## Key Quotes

> "Writers don't want first drafts visible to the editor."

> "I hate Track Changes! I'm dyslexic. I find reading Track Changes nearly impossible."

> "Temporal Paper - Final - PLOS Revisions v3.docx"

## Key Themes

#collaboration #version-control #writing #crdt #ink-and-switch

The "Fishbowl Effect" is the killer insight. Google Docs solved collaboration but created surveillance anxiety. Writers resort to going offline or copying to private files -- defeating the point of collaboration tools entirely. Upwelling addresses this by making private drafts a first-class concept, not a workaround.

The technical foundation is Automerge (CRDT), which means changes are always tracked, conflicts are automatically resolved at the syntactic level, and the system works offline. But the design insight is separating *recording* from *visualization*. Track Changes in Word forces you to opt in to recording; Upwelling always records but lets you choose when to see changes. This inversion is subtle and powerful.

Floating drafts -- where active drafts auto-rebase when another draft merges -- solve the stale-branch problem that plagues Git workflows. The guarantee is strong: what you review is exactly what will merge.

## Critical Analysis

This is classic Ink & Switch: technically deep, beautifully researched, and frustratingly not a shipping product. The research paper format means this influences future tools but doesn't become one itself. Someone needs to build this for real.

The limitation they identify -- CRDTs handle syntactic merges but humans must catch semantic conflicts (like one author changing terminology while another references the old terms) -- is exactly the limitation that LLMs could now address. An AI reviewer that catches semantic drift across drafts would close the loop.

The "named drafts" approach (each draft has a purpose statement) is borrowed from good commit messages in Git. It works because it shifts conflict prevention from merge algorithms to human coordination.

See also [[graphify]] and [[lat.md]] for other approaches to making knowledge navigable, and [[Feedback Loop is All You Need]] for why automated checks beat human discipline.

---
*Sources: [[summary/upwelling]]*
*Last updated: 2026-05-14*
