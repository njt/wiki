# Scour

A solo-dev personalized content feed that uses semantic matching against free-form interest descriptions to surface hidden gems from HN, Reddit, arXiv, Substack, RSS, and thousands of blogs — relevance-ranked rather than popularity-ranked. Built by Evan Schwartz.

---

## What It Is

Scour is a feed reader that inverts the traditional model. Instead of subscribing to specific sources and then filtering by relevance yourself, you describe your interests in free-form text ("Tempeh," "Async Drop," "Non-Proliferation") and Scour does the matching across thousands of sources. A post with 1 upvote can rank above one with 1,000 if it's more semantically relevant to your interests.

The site is built and run by a single person, Evan Schwartz. As of mid-2026 it was scouring ~823K posts/month across 28K community-added sources, surfacing ~65K "hidden gems" for ~3,300 readers.

## Key Mechanics

**Semantic matching, not keyword search.** You write interests as natural language — as broad or niche as you want — and Scour uses embedding-based matching to find relevant articles. This is the core differentiator from RSS readers and Reddit/HN browsing.

**Relevance over popularity.** The ranking is explicitly designed to surface content you'd miss otherwise. A small blog post nobody upvoted can outrank a front-page HN thread if it's more relevant to your interests. This is the mechanism behind the "serendipity machine" claim.

**Keyboard-driven.** Full vim-style navigation (j/k, g-prefixed go-to commands). The design assumes power users who live in the keyboard.

**Weekly digest.** A Friday email summarizes what you might have missed — the "pull" complement to the "push" of the live feed.

**Public feed sharing.** Evan's own feed is publicly viewable, which doubles as both a demo and a social discovery mechanism.

## Why It Matters

Scour sits at the confluence of several interesting trends:

> "It's not a feed reader, it's a serendipity machine." — Zach Charlop-Powers

This quote captures the product ambition: Scour doesn't want to replace your RSS reader, it wants to replace the *discovery* function that social media used to serve before engagement optimization ate it. It's trying to be StumbleUpon for the post-algorithmic era.

> "For the first time since the Stumbleupon days I feel like I'm right on the pulse of things." — Stefan Thomas

The StumbleUpon comparison is telling. That product died in 2018, and nothing really replaced its particular flavor of semi-random, interest-guided discovery. Scour is one of the first credible attempts to rebuild that experience with modern semantic matching instead of collaborative filtering.

> "I read Hacker News daily, but somehow Scour keeps finding gems that I've missed." — Ian Qvist

This is the acid test: can the tool surface things you'd miss even when you're already reading the source material? If it can do that against HN — one of the most heavily-trafficked aggregators — the semantic matching is doing real work.

## Critical Analysis

**The solo-dev bet is both strength and fragility.** Evan Schwartz building this alone means fast iteration and coherent vision (his own feed shows the product is genuinely built for himself). It also means no redundancy — if he burns out or moves on, the service dies. There's no open-source fallback, no export standard, no API. Your curated interest profile lives entirely at his mercy.

**Semantic matching is the right primitive but the wrong moat.** Embedding-based relevance works well enough to feel magical, and it's now cheap enough for a solo dev to run at scale. But it's also commoditized — any competitor can drop in the same models. The moat, if there is one, is the accumulated training signal from user likes/dislikes/saves. Whether Scour has enough users to build that flywheel is an open question.

**The feed reader graveyard is vast.** Google Reader died in 2013. StumbleUpon died in 2018. Countless RSS readers have come and gone. Feed readers are a famously difficult business because the value is personal and the willingness to pay is low. Scour is free for now — the monetization path is unclear, and "free until it isn't" has a grim track record in this category.

**"Hidden gems" is a product thesis, not just a feature.** The entire product is built around the idea that relevance-ranking surfaces better content than popularity-ranking. This is almost certainly true for niche interests, but it's unproven at scale. As the user base grows, the ranking problem gets harder, not easier — more content, more interests, more edge cases in what "relevant" means.

**The keyboard-shortcut density signals the target user.** This isn't a casual consumer product. The vim-style navigation, the g-prefixed go-to commands, the like/dislike/save taxonomy — this is built for people who already have a relationship with information consumption as a craft. That's a small market, but it's also the market that influences what everyone else reads.

**The "no AI slop" signal is becoming a product category.** Danny Aranda's testimonial — "I was mainly just confused there was no AI video slop when I opened the feed" — is darkly funny but also diagnostic. "No AI-generated content" is emerging as a filter criterion the way "no ads" or "no tracking" did before it. Scour doesn't explicitly claim to filter AI slop, but the semantic matching + source curation effectively does.

## Connections

Scour is part of a broader reaction against engagement-optimized feeds. It connects to:

- **[[Spicy Takes Feed]]** — another curated aggregation experiment, but for tech writing specifically rather than general-interest discovery
- **[[Public Domain Image Archive]]** — shares the "discovery over retrieval" UX philosophy; both products treat serendipity as a designed outcome rather than a happy accident
- **[[Dopamine Fracking]]** — the thing Scour is positioning itself against: the industrial extraction of attention through engagement optimization
- **[[The solution might be cancelling my AI subscription (Wilson)]]** — the broader information-diet anxiety that Scour is solving for: how do you find signal in an increasingly noisy information environment?
- **[[The Dead Economy Theory]]** — the "hidden gems" thesis is implicitly a bet that small-scale, human-produced content is worth surfacing against the algorithmic tide; Scour is a practical tool for that bet

The tool also represents an interesting data point in the **[[Smart Models Dumb Pipes]]** pattern: the semantic matching model is smart, but the feed delivery is dumb pipes — a relevance score and a link. No algorithmic timeline optimization, no engagement-maximizing sorting. The model makes one decision (is this relevant?) and the pipe delivers it.

---

*Sources: [[raw/scour]]*
*Last updated: 2026-08-01*
