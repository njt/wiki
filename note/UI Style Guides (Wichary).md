# UI Style Guides (Wichary)

Marcin Wichary's aresluna.org archive preserves 69 UI style guides spanning 1980 to 2026 — from the Amiga Workbench and IBM's CUA through OPEN LOOK, NeXTSTEP, Palm, Symbian, BlackBerry, and the running Apple/Microsoft/Google lineages — as a curated, lightly-restored mirror collection with downloadable bulk archive.

---

A style guide is an argument that consistency is someone's job — usually a design systems team, occasionally a whole platform vendor. What makes this collection remarkable is the *range of institutions* that once took that job seriously: not just Apple and Microsoft, but Sun and AT&T co-authoring OPEN LOOK, an Apple–IBM joint venture (Taligent) shipping its own HIG, General Magic writing "Concepts" for an OS that never shipped at scale, and GO Corporation penning guidelines for PenPoint, a tablet OS decades before tablets were a market. Wichary's glosses do quiet historiography: the Windows guideline gap around 1990 maps exactly onto Microsoft subcontracting its interface thinking to the CUA/IBM partnership.

The dead platforms are the interesting part. Palm's "Zen of Palm," BlackBerry's four successive guideline generations, Symbian Belle, Windows Phone 7's Metro-era interaction guide — these are documents written with full conviction by teams that did not know they were writing for ghosts. They are also, increasingly, the only training data for interface idioms that have no living vendor to maintain them.

## Key quotes

> This is a collection of preserved UI style guides a.k.a. interface guidelines from various companies, starting from the early 1980s and continuing to today.

"Preserved" is the load-bearing word. These documents are corporate output with no institutional reason to survive — vendor sites rot, PDFs vanish. The archive exists because nobody's job includes keeping 1994's CDE certification checklist alive.

> Note a little hole here around 1990, where Microsoft was preoccupied with CUA framework in partnership with IBM.

The best kind of annotation: a gap in the corpus read as an organizational fact. Style guide publication cadence is a fossil record of where a company thought interface authority should live.

> GNOME Human Interface Guidelines / GNOME Human Interface Guidelines / GNOME Human Interface Guidelines

Three identical-titled entries, with the deadpan note that "the versioning number of these guidelines is very confusing." Even the failure modes of style guides are preserved here.

## Key themes

- #concept — the style guide as a genre: platform-level contract between an OS vendor and third-party developers, predating and enabling today's design systems
- #tool — the archive itself: cleaned mirrors plus a one-click bulk download, doing for interface history what software preservation does for games

## Take

This is a source that looks like a link list and functions like a thesis: interface consistency was never just a big-company nicety, it was infrastructure that every failed platform also built, and the failure cases are as instructive as the survivors. Two readings worth pairing:

First, it is raw material for the agent era. When a coding agent builds UI today, its "taste" is trained on exactly these corpora — the skeuomorphic iPhone HIG, Aqua, Material. Wichary's collection is, in effect, a curated slice of the priors. It also slots directly into the practice [[Writing Style Guides for Better UIs]] describes: feed a style guide to an agent and have it enforce the rules across every UI string in seconds, which retroactively makes these 69 documents machine-readable quality gates rather than shelf-ware.

Second, it complicates the design-systems story. [[Design Systems for AI Agents (Luke Wroblewski)]] argues design systems stop rotting when they become part of the AI steering layer — agents read them every time. This archive shows the pre-agent version of the same lifecycle: guidelines that outlived their platforms only because an individual decided to mirror them. Consistency infrastructure has always been one reorganization away from abandonment; the agent steering layer is just the latest attempt to make it stick.

It also strengthens the human-craft side of design-with-agents: [[Turning Your AI Into a World-Class Designer (Chimala)]] argues models need seeded inspiration and subtraction as the human contribution; an archive of 40 years of interface idioms is precisely the seed corpus that argument implies. And it sits next to [[Command Line Interface Guidelines]] as a sibling preservation effort — one for GUIs across platforms, one for a single interface genre, both insisting that interface conventions are worth writing down and keeping.

---
*Sources: [[raw/ui-style-guides]], [[summary/ui-style-guides]]*
*Last updated: 2026-11-26*
