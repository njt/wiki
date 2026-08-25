# BookOrbit

An open-source (AGPL-3.0), self-hosted reading server for ebooks, audiobooks, comics, and PDFs. The pitch is three sentences: your files stay on your hardware, every device stays in sync, and there's no subscription. Under the hood it's a Docker Compose deployment with built-in readers for all four formats, multi-user support with OIDC/SSO, nine-provider metadata management, two-way Kobo/KOReader progress sync, a staging "Book Dock", a private OPDS catalog feed, and reading analytics.

---

## Key Quotes

> "Your files stay on your hardware, and every device stays in sync."

The whole argument in one line, and it's a sovereignty claim, not a features claim. The differentiator isn't "we have a nice reader" — it's that the library itself lives on your disk, not a vendor's cloud. Sync is offered as the payoff for staying local, which quietly refutes the usual assumption that cloud storage is the price of cross-device access.

> "One Compose file, local storage, no subscription."

Deployment and pricing collapsed into a single bullet. "No subscription" is doing political work: it's a rejection of the rent-your-library model that Kindle, Audible, and every streaming-adjacent reading service run on.

> "Private feed for KOReader, Thorium, Moon+, and more."

The most telling feature is the one about other people's apps. BookOrbit doesn't try to win by owning the client — it exports an OPDS catalog so a half-dozen existing readers can consume it. Interoperability over lock-in.

## Key Themes

#tool #self-hosting #open-source #books #local-first

- **Self-hosting as library sovereignty.** The same "your data never touches someone else's servers" logic that drives [[Headscale]] and [[Self-Hosted LLMs]], applied to the one collection people have actually paid for over decades. Your ebooks are the last media you can trivially own as files; BookOrbit bets you'd rather serve them yourself than keep them in a Kindle account that can be revoked.
- **Interoperability as the moat.** Kobo sync, KOReader sync, and an OPDS feed make BookOrbit a *server* for an existing ecosystem, not a walled garden. The opposite of Amazon's Kindle strategy.
- **Metadata as the hard problem.** Nine providers, bulk editing, field-level rules, and a staging "Book Dock" for reviewing metadata before it goes live. The people who build these tools know the actual pain of a personal library isn't reading — it's cataloguing. This rhymes with the wiki's repeated finding that the boring, human-taste-adjacent work is the part that doesn't automate away.
- **The anti-subscription reflex.** "No subscription" as a first-class selling point, in the same spirit as the fair-code/self-host positioning of [[n8n]]. For a certain user, the license and the hosting model are the feature.

## Critical Analysis

**What's strong:** The scope is coherent. BookOrbit isn't trying to be an LLM or a social network; it's a focused self-hosted media server for one category (text-based media) with exactly the features that category's power users want — OPDS, Kobo/KOReader sync, field-level metadata rules. Choosing AGPL-3.0 rather than fair-code signals the project wants to be genuinely open source, not merely source-available.

**What's interesting:** The ereader-sync feature is subtle and important. The hardest thing about self-hosting your library is that your actual reading device (a Kobo) is a vendor device with its own cloud. Two-way progress sync with Kobo and KOReader bridges that gap — your library is yours, but your reading position still follows you onto hardware you didn't build. That's the realistic, unglamorous version of "self-hosted" that pure-ideology accounts miss.

**What's missing:** The landing page is a feature grid, not documentation. There's no roadmap, no statement of DRM handling (can it read the encrypted EPUBs you bought from Kobo, or only DRM-free files?), and no position on which metadata providers need API keys. The Kobo-sync claim in particular hides a hard integration problem — Kobo's cloud sync isn't an open protocol, so this either piggybacks on Kobo's API or works only with sideloaded books. Whether the whole thing survives a vendor API change is the same structural bet [[Headscale]] makes against Tailscale.

**Where it fits:** BookOrbit is the reading-library node in the wiki's self-hosting thread — [[Headscale]] (network), [[PiClaw]] and [[Odysseus]] (agents), [[n8n]] (workflows), and now books. It also sharpens the contrast with [[1lib]]: shadow libraries solve *access* to books by hosting them for everyone, illegally; BookOrbit solves *ownership* by hosting the books you already have, for yourself. Two different answers to the same question of who holds the library.

---
*Sources: [[raw/bookorbit-app]], [[summary/bookorbit-app]]*
*Last updated: 2026-08-25*
