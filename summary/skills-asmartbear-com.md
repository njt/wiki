---
url: https://skills.asmartbear.com/
title: "A Smart Bear Skills"
author: Jason Cohen
date_fetched: 2026-08-21
date_published: 2026-08-16
topics:
  - claude-code
  - ai-product-and-business
---

Jason Cohen — founder of two unicorns (WP Engine, Smart Bear) and author of *Hidden Multipliers* — has packaged the frameworks from his A Smart Bear blog into a set of Claude Code skills ("asb-skills"). The site advertises both multi-step **workshops** (Find Yourself, Find Your Carol, Customer Interviews, Set Your Price) and **standalone** skills (Rude Q&A, Good Market?, Positioning, Needs Stack), all installed via `npx skills add asmartbear/asb-skills` (or as a Claude Code plugin, or a ZIP).

The defining philosophy is stated up front: the skills don't think for you. Each one interrogates you — "pressing on vague claims and refusing to let you off the hook for a weak answer" — the way Cohen would if he were facilitating in the room. You leave with your own best thinking, not the AI's. This is grounded in a psychological claim repeated across Cohen's articles: you cannot interrogate yourself, because denial and excuse-making are normal human behavior, which is why strategy documents are full of hope instead of the scary challenges the business really faces. An outside attacker is the only reliable way past it.

Two skills are documented in depth. **Rude Q&A: The Constructive Devil's Advocate** (`asb-rude-qa`) turns Claude into that attacker: it asks unfair questions, refuses vague answers, and stays on each point until you give a real answer. You get one of three honest results — a sharper plan with consequences accepted, a list of what you still don't know, or the conclusion the idea was wrong. The exercise is framed as practice with a "heavy bat": you face attacks harder than reality will bring, so the real thing feels easier.

**Good Market?** (`asb-problem`) is a seven-factor Fermi scorecard for business viability, distilled from the article "Excuse me, is there a problem?" It scores a specific target market on seven multiplying, power-of-ten criteria — Plausible, Self-Aware, Lucrative, Liquid, Eager (identity), Eager (comparative), Enduring — each backed by its own article (Product Purgatory, willingness-to-pay, leverage, "Worse, but unique", pricing, "Selling to Carol"). The multiplied score is explicitly directional, not precise: it shows whether your big strengths overcome your few weaknesses, and when negative, pushes you toward a narrower niche before concluding the idea isn't viable.

The site is a Starlight-built docs site, and the raw fetch captures the page's full CSS and JS alongside the prose (the `content:` fields are the extracted text).
